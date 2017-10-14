---
layout:     post
title:      在Android应用中实现Email发送
category: blog
description: 之前在开发Java时曾经写过发email的代码，本以为复用过来就可以，但是发现比想象中复杂的多~
---

## 在Android应用中实现Email发送

在开发APP过程中有一个用邮件发送验证码的需求，所以就研究了一下这方面的东西，主要遇到了如下四种BUG，总结如下：

#### 1.java.lang.NoClassDefFoundError: javax.activation.DataHandler

我把原来实现的Java发送email程序放进来之后，运行App发生了这个bug，查找问题之后，一个博客介绍了Android中的email实现与Java中有区别，所以我使用了另一种写法，但是发现依然是这个bug。

然后，在stackoverflow上看到一个回答，安卓App中除了mail.jar外，还要引入activation.jar和additionnal.jar，这也是最普遍的一种解决方法，但是我把这两个下载导入之后发现依然没有解决，现在分析可能是版本问题导致的。

最后，还是在stackoverflow上找到了另一个解决办法，在app的gradle中添加两个官网的依赖，从而解决这一问题。所添加的依赖如下：

```
compile 'com.sun.mail:android-mail:1.5.5'
compile 'com.sun.mail:android-activation:1.5.5'
```

#### 2.NetworkOnMainThreadException异常

在解决了上一个问题之后，又遇到了这个问题，从名字上就可以看出来与网络有关的处理都不能在主线程中运行，利用创建一个线程，然后调用这个线程解决了这个问题。

代码如下：

```
	private Handler eHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            // UI界面的更新等相关操作
        }
    };

    private Runnable emailTask = new Runnable() {

        @Override
        public void run() {
            // 在这里进行 http request.网络请求相关操作
            androidSendEmail ase = new androidSendEmail();
            try {
                ase.sendEmail("1350763025@qq.com","HiClass注册验证码","你的验证码是"+createActiveCode);
            } catch (Exception e) {
                e.printStackTrace();
            }
            Message msg = new Message();
            handler.sendMessage(msg);
        }
    };
```

调用方法：

```
new Thread(emailTask).start();
```

#### 3.java.lang.SecurityException:Permission denied

查找之后得到解决方法：需要在AndroidManifest清单文件中声明需要用到的Internet权限，在AndroidManifest.xml中与Application平级的地方添加如下配置即可：

```
<uses-permission android:name="android.permission.INTERNET"></uses-permission>
```

#### 4.javax.mail.AuthenticationFailedException

最后遇到了这个问题，这个问题在实现发送email时会经常遇到，就是验证身份时出了问题，检查发送email的代码发现邮箱写错了，改正之后完成发送邮件功能。

虽然一个小东西搞了一下午，遇到这么多bug，但是感觉还是挺有意思的~~~

最后附上发送email部分的代码：

```
public class androidSendEmail {
    /**
     * 邮件发送程序
     *
     * @param to
     *            接受人
     * @param subject
     *            邮件主题
     * @param content
     *            邮件内容
     * @throws Exception
     * @throws MessagingException
     */
    public static void sendEmail(String to, String subject, String content) throws Exception, MessagingException {
        String host = "smtp.qq.com";
        String address = "1350763025@qq.com";
        String from = "1350763025@qq.com";
        String password = "yang19950206";// 密码
        if ("".equals(to) || to == null) {
            to = "545099227@qq.com";
        }
        String port = "25";
        SendEmail(host, address, from, password, to, port, subject, content);
    }

    /**
     * 邮件发送程序
     *
     * @param host
     *            邮件服务器 如：smtp.qq.com
     * @param address
     *            发送邮件的地址 如：545099227@qq.com
     * @param from
     *            来自： wsx2miao@qq.com
     * @param password
     *            您的邮箱密码
     * @param to
     *            接收人
     * @param port
     *            端口（QQ:25）
     * @param subject
     *            邮件主题
     * @param content
     *            邮件内容
     * @throws Exception
     */
    public static void SendEmail(String host, String address, String from, String password, String to, String port, String subject, String content) throws Exception {
        Multipart multiPart;
        String finalString = "";

        Properties props = System.getProperties();
        props.put("mail.smtp.starttls.enable", "true");
        props.put("mail.smtp.host", host);
        props.put("mail.smtp.user", address);
        props.put("mail.smtp.password", password);
        props.put("mail.smtp.port", port);
        props.put("mail.smtp.auth", "true");
        Log.i("Check", "done pops");
        Session session = Session.getDefaultInstance(props, null);
        DataHandler handler = new DataHandler(new ByteArrayDataSource(finalString.getBytes(), "text/plain"));
        MimeMessage message = new MimeMessage(session);
        message.setFrom(new InternetAddress(from));
        message.setDataHandler(handler);
        Log.i("Check", "done sessions");

        multiPart = new MimeMultipart();
        InternetAddress toAddress;
        toAddress = new InternetAddress(to);
        message.addRecipient(Message.RecipientType.TO, toAddress);
        Log.i("Check", "added recipient");
        message.setSubject(subject);
        message.setContent(multiPart);
        message.setText(content);

        Log.i("check", "transport");
        Transport transport = session.getTransport("smtp");
        Log.i("check", "connecting");
        transport.connect(host, address, password);
        Log.i("check", "wana send");
        transport.sendMessage(message, message.getAllRecipients());
        transport.close();
        Log.i("check", "sent");
    }
}
```

