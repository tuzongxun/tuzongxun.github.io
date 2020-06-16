---
title: springboot2集成oauth2和keycloak以及admin rest api记录
date: 2019-8-8 9:43:58
categories: java #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
tags: [keycloak,java]
---
# 前言
以keycloak作为sso认证中心服务端，springboot2的客户端集成方式有很多种，例如仅集成keycloak的jar包方式、集成spring security的方式、以及security+oauth2的方式等。
上述三种方式，从实现以及功能上来说均是一个比一个复杂。
另外，springboot作为普通客户端的同时，也可以进行更多的集成，进而实现对keycloak服务端的操作，这就涉及到keycloak中admin rest api的调用。
正常而言，rest api符合rest规范，应该是比较简单的。但是当rest api牵扯到各种权限和角色的时候，会发现很多其他的细节问题会导致这个rest接口无法调通，尤其是这些问题不是代码本身问题的时候，就会更加让人摸不着头脑。
以下是初步集成security+oauth2+admin rest api过程中部分踩坑记录，其中有很多细节还有待深入理解。
<!--more-->

# security+oauth2客户端集成
## 条件说明
客户端集成的前提是，有了已经可用的keycloak服务端，并且已经在服务端控制台创建好了realm、
client、role、scope等，可以参考上一篇：
https://tuzongxun.blog.csdn.net/article/details/96979245

## 环境说明
spingboot1.x和springboot2.x集成keycloak的方式是有一定差别的，鉴于为实际项目服务的宗旨，这一次的集成预研，基于springboot2.1.3版本，以下是客户端集成时的maven依赖配置：
<pre>
&lt;dependency>
&lt;groupId>org.springframework.boot&lt;/groupId>
&lt;artifactId>spring-boot-starter-web&lt;/artifactId>
&lt;/dependency>
&lt;dependency>
&lt;groupId>org.springframework.boot&lt;/groupId>
&lt;artifactId>spring-boot-starter-oauth2-client&lt;/artifactId>
&lt;/dependency>
&lt;dependency>
&lt;groupId>org.springframework.boot&lt;/groupId>
&lt;artifactId>spring-boot-starter-oauth2-resource-server&lt;/artifactId>
&lt;/dependency> 
&lt;dependency>
&lt;groupId>org.springframework.boot&lt;/groupId>
&lt;artifactId>spring-boot-starter-security&lt;/artifactId>
&lt;/dependency>
&lt;dependency>
&lt;groupId>org.springframework.security.oauth&lt;/groupId>
&lt;artifactId>spring-security-oauth2&lt;/artifactId>
&lt;version>2.1.3.RELEASE&lt;/version>
&lt;/dependency>
&lt;dependency>
&lt;groupId>org.springframework.security&lt;/groupId>
&lt;artifactId>spring-security-oauth2-jose&lt;/artifactId>
&lt;/dependency>
&lt;dependency>
&lt;groupId>org.springframework.cloud&lt;/groupId>
&lt;artifactId>spring-cloud-starter-oauth2&lt;/artifactId>
&lt;version>2.1.3.RELEASE&lt;/version>
&lt;/dependency>
&lt;dependency>
&lt;groupId>org.springframework.cloud&lt;/groupId>
&lt;artifactId>spring-cloud-starter-security&lt;/artifactId>
&lt;version>2.1.3.RELEASE&lt;/version>
&lt;/dependency>
</pre>

## 项目配置说明
oauth2授权验证时，需要token等令牌，sso单点登录需要一个统一的登录入，这些均是keycloak服务端提供，因此就必须在客户端集成时进行oauth2的配置，各种url指向对应的keycloak服务的url，如下：
<pre>
server:
  port: 8884 
spring:
  application:
    name: oauthdemo
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://127.0.0.1:8080/auth/realms/tzx
          jwk-set-uri: http://127.0.0.1:8080/auth/realms/tzx/protocol/openid-connect/certs
      client:
        provider:
          master:
            issuer-uri: http://127.0.0.1:8080/auth/realms/tzx
            token-uri: http://127.0.0.1:8080/auth/realms/tzx/protocol/openid-connect/token
            authorization-uri: http://127.0.0.1:8080/auth/realms/tzx/protocol/openid-connect/auth
            user-info-uri: http://127.0.0.1:8080/auth/realms/tzx/protocol/openid-connect/userinfo
            jwk-set-uri: http://127.0.0.1:8080/auth/realms/tzx/protocol/openid-connect/certs
            user-info-authentication-method: header
            user-name-attribute: preferred_username
        registration:
          master:
            client-id: test-authz
            client-secret: 2811de6b-e703-4644-8330-617ac5104ca6
            client-name: test-authz
            provider: master
            authorization-grant-type: authorization_code
            client-authentication-method: basic
            scope:
              - email
              - profile
              - openid
              - openid_test
</pre>

上边内容需要注意的是：
1. 大部分内容对可以照搬的，仅需改一下ip端口和realm
2. 各个url里边realms后边的tzx实际上就是在keycloak服务端创建的realm，这里的tzx就是我自己创建的一个realm。
3. client-id、client-secret、client-name这些很多资源有讲，有示例的，就不多说。
4. scope里边配置的是一个数组，这里配了四个，实际上前两个是keycloak我们创建客户端时就默认会有的，后两个是需要自己创建的。在本次集成中，实际也只有最后一个起了作用，会看到后边的代码中有用到。

##  WebSecurity配置类
集成了security，并且要启用的话，就需要根据实际重写security适配器，简易代码如下：
<pre>
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
@Configuration
@EnableWebSecurity
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests().antMatchers("/test/*").hasRole("USER").antMatchers("/token")		.hasAuthority("SCOPE_openid_test").anyRequest().authenticated().and().oauth2Login().and()
				.oauth2Client().and().oauth2ResourceServer().jwt();
	}
}
</pre>
这里边需要注意的就是SCOPE_openid_test_api，这个实际上就是上边说的那个scope，不过呢直到这里，配置文件中的scope配置其实都还是没有起作用的，配置文件中的那个配置是在后边有用。这里生效的就是代码里所写的，需要保证这个scope在keycloak服务端有创建。

## OAuth2Client的配置
我们启用了oauth2验证之后，在各个接口就需要token等令牌信息，只有令牌校验通过，这个接口才应该被正常的访问。
那么访问接口如何携带令牌等授权信息，oauth2对restTemplat进行了封住，需要我们使用的时候进行一定的处理，以使它知道如何获取令牌如何携带令牌，初步实现代码如下：
<pre>
import java.util.ArrayList;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.client.DefaultOAuth2ClientContext;
import org.springframework.security.oauth2.client.OAuth2ClientContext;
import org.springframework.security.oauth2.client.OAuth2RestTemplate;
import org.springframework.security.oauth2.client.registration.ClientRegistration;
import org.springframework.security.oauth2.client.registration.ClientRegistrationRepository;
import org.springframework.security.oauth2.client.resource.OAuth2ProtectedResourceDetails;
import org.springframework.security.oauth2.client.token.AccessTokenRequest;
import org.springframework.security.oauth2.client.token.DefaultAccessTokenRequest;
import org.springframework.security.oauth2.client.token.grant.client.ClientCredentialsResourceDetails;
import org.springframework.security.oauth2.common.AuthenticationScheme;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableOAuth2Client;
@Configuration
@EnableOAuth2Client
public class OAuth2ClientConfiguration {
	@Autowired
	private ClientRegistrationRepository clientRegistrationRepository;
	@Bean
	protected OAuth2ProtectedResourceDetails resource() {
		ClientCredentialsResourceDetails resource = new ClientCredentialsResourceDetails();
		ClientRegistration clientRegistration = clientRegistrationRepository.findByRegistrationId("master");
		resource.setAccessTokenUri(clientRegistration.getProviderDetails().getTokenUri());
		resource.setClientId(clientRegistration.getClientId());
		resource.setClientSecret(clientRegistration.getClientSecret());
		resource.setClientAuthenticationScheme(AuthenticationScheme.header);
		resource.setClientAuthenticationScheme(AuthenticationScheme.header);
		List<String> scopes = new ArrayList<String>(clientRegistration.getScopes());
		resource.setScope(scopes);

		return resource;
	}
	@Bean
	public OAuth2RestTemplate restTemplate() {
		AccessTokenRequest accessTokenRequest = new DefaultAccessTokenRequest();
		OAuth2ClientContext oAuth2ClientContext = new DefaultOAuth2ClientContext(accessTokenRequest);
		return new OAuth2RestTemplate(resource(), oAuth2ClientContext);
	}
}
</pre>

上边代码主要作用就是为了装配OAuth2RestTemplate这个bean，以用来在后边发送oauth2授权的请求，下边就是一个相应的controller。

## 测试示例
<pre>
@Autowired
private OAuth2RestTemplate restTemplate;
@GettMapping( "/test/getToken")
public String client() {
		String result = restTemplate.postForObject("http://127.0.0.1:8884/token", null, String.class);
		return result;
}
@PostMapping("/token")
public String getToken(@AuthenticationPrincipal Jwt jwt) {
		String tokenId = jwt.getId();
		String value = jwt.getTokenValue();
		return "tokenId:" + tokenId + ",token:" + value;
}
</pre>

这里为何要写两个接口呢？仔细看便会发现，第一个接口就是一个普通接口，除开收到之前security的权限限制外，就没有别的条件，因此这个接口是可以不用任何额外操作，可以直接在浏览器地址栏请求的。
在这个普通接口里做了二次请求，目标接口用了@AuthenticationPrincipal注解以及jwt令牌类，里边的逻辑就是获取tokenid和token内容。
可能有人已经明白了，之所以这里一个注解和一个特定类就能拿到token，就是因为上边的restTemplate使用的是我们配置过的OAuth2RestTemplate，他在发请求的时候就会去配置文件中查找资源，请求token，然后再发起实际的目标请求。进而在目标接口收到请求的时候就可以根据注解和特定的类拿到token等信息。
到这里，springboot2+secutiry+oauth2+jwt+keycloak的一个基本集成就算是完成了，浏览器访问http://localhost:8884/test/getToken就会返回token等信息。

# admin rest api调用
在上边的例子中，我们有配置client-id等信息，这些信息均来自与keycloak服务端。网上的各种示例说明，基本都是说的直接在keycloak服务端控制台创建，也就是跟我们用任何软件一样点点鼠标。
而实际上，keycloak提供了admin rest api，以使我们可以再java代码中调用，来创建各种原本在keycloak控制台创建的资源，下边就以创建一个简单的client作为示例进行说明。

## 依赖集成
java操作keycloak服务端，这个java代码所在的项目实际上本身就是一个客户端，因此上边的那些依赖、配置和代码其实也都是必要的，同时，除了上边的依赖外，还需要另外集成两个依赖：
<pre>
&lt;dependency>
&lt;groupId>org.keycloak&lt;/groupId>
&lt;artifactId>keycloak-authz-client&lt;/artifactId>
&lt;version>6.0.1&lt;/version>
&lt;/dependency>
&lt;dependency>
&lt;groupId>org.keycloak&lt;/groupId>
&lt;artifactId>keycloak-admin-client&lt;/artifactId>
&lt;version>6.0.1</version>
&lt;/dependency>
</pre>

## 代码示例
需要再次声明的是，java操作keycloak服务端，这个java代码所在的项目实际上本身就是一个客户端，因此上边的那些依赖、配置和代码其实也都是必要的。
在上边的基础上，如果需要用java代码创建一个client，示例代码如下：
<pre>
@RequestMapping("/test/createClient")
	public void test() {
		try {
			ClientRepresentation client = new ClientRepresentation();
			client.setClientId("client-test000123");
			client.setId("client-test-id00123");
			client.setPublicClient(false);
			client.setSecret("1235879");
			client.setEnabled(true);
			client.setProtocol("openid-connect");
			List<String> origins = new ArrayList<String>();
			origins.add("*");
			client.setWebOrigins(origins);
			List<String> urls = new ArrayList<String>();
			urls.add("*");
			client.setRedirectUris(urls);
			client.setClientAuthenticatorType("client-secret");
			client.setServiceAccountsEnabled(true);
			client.setDirectAccessGrantsEnabled(true);
			HttpHeaders headers = new HttpHeaders();
			headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
			HttpEntity<?> httpEntity = new HttpEntity<>(client, headers);
			String jsonStr;
			restTemplate.postForObject("http://127.0.0.1:8080/auth/admin/realms/tzx/clients", httpEntity,String.class, client);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
</pre>

上边的代码主要是参考keycloak官网的admin rest api操作说明：
https://www.keycloak.org/docs-api/6.0/rest-api/
代码是没有问题的，但是如果有人从上往下抄一遍，会发现执行上边的创建客户端的请求会报错：
<pre>
org.springframework.web.client.HttpClientErrorException$Forbidden: 403 Forbidden
	at org.springframework.web.client.HttpClientErrorException.create(HttpClientErrorException.java:83)
	at org.springframework.web.client.DefaultResponseErrorHandler.handleError(DefaultResponseErrorHandler.java:122)
	at org.springframework.web.client.DefaultResponseErrorHandler.handleError(DefaultResponseErrorHandler.java:102)
</pre>

为何会这样呢？这其实就是我说的坑，原因在与当前这个client的role里没有创建client的权限。
例如我这里配置文件里的client是faw-api-authz，那么要想成功用java代码创建别的client
，就需要faw-api-authz拥有创建client的权限，需要在keycloak控制台设置步骤依次如下：
进入tzx这个realm——》点击clients——》找到并点击faw-api-authz——》点击service Account roles——》找到client roles——》点击下拉框，找到realm management——》选择create client和manage clients——》点击add select。
注：settings那里的service accounts enable需要打开。
有了上边的设置之后，重启springboot服务，再次访问，会发现我们新的client就创建成功了，在keycloak的 控制台的clients也会看到多了一个，各种参数就是java代码里写的参数。
至此，springboot2集成security+oauth2+jwt+keycloak+keycloak admin rest api基本完成，各种细节性的配置和选择需要在此基础上进一步优化。
