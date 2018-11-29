---
title:  JAVA代码发送邮件示例和解释(二)
date: 2017-01-30 15:51:52
tags: [java,邮件]
categories: [java,java] #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
---
之前有使用过一次在程序中发邮件的功能，也写了一篇相关的笔记，当时使用的是163邮箱，经过测试多个163账号都是可行的。但是最近把账号换成中国移动@chinamobilesz.com这种结尾的时候，却一直连接不上服务器，验证不通过，于是只好重新进行了一些改动，这期间也有参考另一个同事之前的写法，成功实现。
<!--more-->
这种实现方式和上一种大同小异，只是经过测试后，这种方式不仅163的邮箱没有问题，中国移动@chinamobilesz.com这种也可以，因此相对前边写的那种应该要更好一些，具体代码如下：
<pre>
package test;   
import java.util.ArrayList;  
import java.util.Date;  
import java.util.List;  
import java.util.Properties;  
import java.util.regex.Matcher;  
import java.util.regex.Pattern;  
import javax.activation.DataHandler;  
import javax.activation.DataSource;  
import javax.activation.FileDataSource;  
import javax.mail.Address;  
import javax.mail.BodyPart;  
import javax.mail.Message;  
import javax.mail.Session;  
import javax.mail.Transport;  
import javax.mail.internet.InternetAddress;  
import javax.mail.internet.MimeBodyPart;  
import javax.mail.internet.MimeMessage;  
import javax.mail.internet.MimeMultipart;  
import javax.mail.internet.MimeUtility;   
public class SendMailTest1 {  
    public static void main(String[] args) {  
        SendMailTest1 send = new SendMailTest1();  
        send.sendEmail();  
    }  
    /**   
     *@Title: sendTextMail 
     *@Description: TODO 
     *@param mailInfo 
     *@return 
     */  
    public boolean sendEmail() {  
        // 从配置文件中读取配置信息  
        Properties pro = new Properties();  
        pro.put("mail.smtp.host", "mail.chinamobilesz.com");  
        pro.put("mail.smtp.auth", "true");  
        // Properties pro = mailConfig.getProperties();  
        // 根据邮件的回话属性构造一个发送邮件的Session  
        MailAuthenticator authenticator = new MailAuthenticator("账号",  
                "密码");  
        Session session = Session.getInstance(pro, authenticator);  
        // 监控邮件命令  
        try {  
            // 根据Session 构建邮件信息  
            Message message = new MimeMessage(session);  
            // 创建邮件发送者地址  
            Address from = new InternetAddress("xtyw");  
            // 设置邮件消息的发送者  
            message.setFrom(from);  
            // 验证邮箱地址  
            List<String> auth = new ArrayList<String>();  
            auth.add("1160569243@qq.com");  
            String toAddress = validateEmail(auth);  
            if (!toAddress.isEmpty()) {  
                // 创建邮件的接收者地址  
                Address[] to = InternetAddress.parse(toAddress);  
                // 设置邮件接收人地址  
                message.setRecipients(Message.RecipientType.TO, to);  
                message.setSubject("12345");  
                // 邮件容器  
                MimeMultipart mimeMultiPart = new MimeMultipart();  
                // 设置HTML  
                BodyPart bodyPart = new MimeBodyPart();  
                String htmlText = "123456";  
                bodyPart.setContent(htmlText, "text/html;charset=utf-8");  
                mimeMultiPart.addBodyPart(bodyPart);  
                // 添加附件  
                List<String> fileList = new ArrayList<String>();  
                fileList.add("C:\\Users\\tuzongxun123\\Desktop\\自主服务API.docx");  
                if (fileList != null) {  
                    BodyPart attchPart = null;  
                    for (int i = 0; i < fileList.size(); i++) {  
                        if (!fileList.get(i).isEmpty()) {  
                            attchPart = new MimeBodyPart();  
                            // 附件数据源  
                            DataSource source = new FileDataSource(  
                                    fileList.get(i));  
                            // 将附件数据源添加到邮件体  
                            attchPart.setDataHandler(new DataHandler(source));  
                            // 设置附件名称为原文件名  
                            attchPart.setFileName(MimeUtility.encodeText(source  
                                    .getName()));  
                            mimeMultiPart.addBodyPart(attchPart);  
                        }  
                    }  
                }  
                message.setContent(mimeMultiPart);  
                message.setSentDate(new Date());  
                // 保存邮件  
                message.saveChanges();  
                // 发送邮件  
                Transport.send(message);  
                return true;  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
            System.out.println("邮件发送失败");  
        }  
        return false;  
    }  
    /** 
     *@title validateEmail 
     *@Description 验证邮箱格式 
     *@param emailList 
     *@return string 
     */  
    public String validateEmail(List<String> emailList) {  
        StringBuffer buffer = new StringBuffer();  
        if (!emailList.isEmpty()) {  
            String regEx = "^([a-z0-9A-Z]+[-|\\.]?)+[a-z0-9A-Z]@([a-z0-9A-Z]+(-[a-z0-9A-Z]+)?\\.)+[a-zA-Z]{2,}$";  
            Pattern p = Pattern.compile(regEx);  
            for (int i = 0; i < emailList.size(); i++) {  
                Matcher match = p.matcher(emailList.get(i));  
                if (match.matches()) {  
                    buffer.append(emailList.get(i));  
                    if (i < emailList.size() - 1) {  
                        buffer.append(",");  
                    }  
                }  
            }  
        }  
        return buffer.toString();  
    }  
}  
</pre>

自定义邮箱权限信息类：
<pre>
public class MailAuthenticator extends Authenticator {  
  /**  
  *用户名  
  */  
  private String username;  
  /**  
  *密码  
  */  
  private String password;  
  /**  
   *创建一个新的实例 MailAuthenticator.    
   *@param username  
   *@param password  
   */  
  public MailAuthenticator(String username, String password) {  
    this.username = username;  
    this.password = password;  
  }  
  public String getPassword() {  
    return password;  
  }  
  @Override  
  protected PasswordAuthentication getPasswordAuthentication() {  
    return new PasswordAuthentication(username, password);  
  }  
  public String getUsername() {  
    return username;  
  }  
  public void setPassword(String password) {  
    this.password = password;  
  }  
  public void setUsername(String username) {  
    this.username = username;  
  }  
}  
</pre>