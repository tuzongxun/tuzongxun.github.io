---
title: JAVA代码发送邮件示例和解释笔记
date: 2017-01-30 15:43:55
tags: [java,邮件]
categories: [java,java] #分类
toc: true  #在此处设定是否开启目录，需要主题支持。
---
下载和上传附件、发送短信和发送邮件，都算是程序中很常用的功能，之前记录了文件的上传和下载还有发送短信，由于最近比较忙，邮件发送的功能就没有时间去弄，好在昨晚终于走通代码成功以163邮箱发送邮件到qq邮箱，以下是相关代码，具体解释可以参考代码中注释：
<!--more-->
<pre>
package test;  
import java.util.ArrayList;  
import java.util.Date;  
import java.util.List;  
import java.util.Properties;  
import java.util.regex.Matcher;  
import java.util.regex.Pattern;  
import javax.mail.Address;  
import javax.mail.Authenticator;  
import javax.mail.BodyPart;  
import javax.mail.Message;  
import javax.mail.PasswordAuthentication;  
import javax.mail.Session;  
import javax.mail.Transport;  
import javax.mail.internet.InternetAddress;  
import javax.mail.internet.MimeBodyPart;  
import javax.mail.internet.MimeMessage;  
import javax.mail.internet.MimeMultipart;  
import com.sun.mail.util.MailSSLSocketFactory;   
/**    
 *@author tuzongxun123  
 *@Description 邮件发送测试类  
 */  
public class sendMailTest {  
   public static void main(String[] args) throws Exception {  
        // 配置信息  
        Properties pro = new Properties();  
        pro.put("mail.smtp.host", "smtp.163.com");  
        pro.put("mail.smtp.auth", "true");  
        // SSL加密  
        MailSSLSocketFactory sf = null;  
        sf = new MailSSLSocketFactory();  
        // 设置信任所有的主机  
        sf.setTrustAllHosts(true);  
        pro.put("mail.smtp.ssl.enable", "true");  
        pro.put("mail.smtp.ssl.socketFactory", sf);  
        // 根据邮件的会话属性构造一个发送邮件的Session，这里需要注意的是用户名那里不能加后缀，否则便不是用户名了  
        //还需要注意的是，这里的密码不是正常使用邮箱的登陆密码，而是客户端生成的另一个专门的授权码  
        MailAuthenticator authenticator = new MailAuthenticator("tuzongxun123",  
                "客户端授权码");  
        Session session = Session.getInstance(pro, authenticator);  
        // 根据Session 构建邮件信息  
        Message message = new MimeMessage(session);  
        // 创建邮件发送者地址  
        Address from = new InternetAddress("tuzongxun123@163.com");  
        // 设置邮件消息的发送者  
        message.setFrom(from);  
        // 验证收件人邮箱地址  
        List<String> toAddressList = new ArrayList<>();  
        toAddressList.add("1160569243@qq.com");  
        StringBuffer buffer = new StringBuffer();  
        if (!toAddressList.isEmpty()) {  
            String regEx = "^([a-z0-9A-Z]+[-|\\.]?)+[a-z0-9A-Z]@([a-z0-9A-Z]+(-[a-z0-9A-Z]+)?\\.)+[a-zA-Z]{2,}$";  
            Pattern p = Pattern.compile(regEx);  
            for (int i = 0; i < toAddressList.size(); i++) {  
                Matcher match = p.matcher(toAddressList.get(i));  
                if (match.matches()) {  
                    buffer.append(toAddressList.get(i));  
                    if (i < toAddressList.size() - 1) {  
                        buffer.append(",");  
                    }  
                }  
            }  
        }  
        String toAddress = buffer.toString();  
        if (!toAddress.isEmpty()) {  
            // 创建邮件的接收者地址  
            Address[] to = InternetAddress.parse(toAddress);  
            // 设置邮件接收人地址  
            message.setRecipients(Message.RecipientType.TO, to);  
            // 邮件主题  
            // message.setSubject("java邮件测试");  
            message.setSubject("为什么错了");  
            // 邮件容器  
            MimeMultipart mimeMultiPart = new MimeMultipart();  
            // 设置HTML  
            BodyPart bodyPart = new MimeBodyPart();  
            // 邮件内容  
            // String htmlText = "java邮件测试111";  
            String htmlText = "为什么错了";  
            bodyPart.setContent(htmlText, "text/html;charset=utf-8");  
            mimeMultiPart.addBodyPart(bodyPart);  
          // 添加附件  
            List<String> fileAddressList = new ArrayList<String>();  
            fileAddressList  
                    .add("C:\\Users\\tuzongxun123\\Desktop\\新建 Microsoft Office Word 文档.docx");  
            if (fileAddressList != null) {  
                BodyPart attchPart = null;  
                for (int i = 0; i < fileAddressList.size(); i++) {  
                    if (!fileAddressList.get(i).isEmpty()) {  
                        attchPart = new MimeBodyPart();  
                        // 附件数据源  
                        DataSource source = new FileDataSource(  
                                fileAddressList.get(i));  
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
        }  
    }  
}   
class MailAuthenticator extends Authenticator {  
     //用户名   
    private String username;   
     //密码   
    private String password;  
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


注：我有个同事使用我这个代码更换为他的账号和客户端授权码后，一运行就报错，然后重置了一下邮箱的客户端授权码后，错误便消失了。