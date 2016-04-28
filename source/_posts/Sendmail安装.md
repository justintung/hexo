title: Sendmail安装
date: 2016-04-23 22:14:36
tags:
- sendmail
- 安装
---
假设：服务器IP为：180.76.149.123，所在内网IP:**192.168.0**.123
###A. 安装相关软件包
centos默认安装了sendmail服务，但还需要安装相关软件包
```shell
yum -y install sendmail sendmail-cf cyrus-sasl
```
cyrus-sasl用于smtp认证
###B.添加邮件用户
邮件用户使用的是系统用户
~~~shell
useradd service
passwd service
~~~
修改密码为：qweasd123
###C.配置
1./etc/mail/sendmail.mc配置
```shell
……
dnl # PLAIN is the preferred plaintext authentication method and used by
dnl # Mozilla Mail and Evolution, though Outlook Express and other MUAs do
dnl # use LOGIN. Other mechanisms should be used if the connection is not
dnl # guaranteed secure.
dnl # Please remember that saslauthd needs to be running for AUTH. 
dnl #
dnl TRUST_AUTH_MECH(`EXTERNAL DIGEST-MD5 CRAM-MD5 LOGIN PLAIN')dnl
dnl define(`confAUTH_MECHANISMS', `EXTERNAL GSSAPI DIGEST-MD5 CRAM-MD5 LOGIN PLAIN')dnl
TRUST_AUTH_MECH(`EXTERNAL DIGEST-MD5 CRAM-MD5 LOGIN PLAIN')dnl
define(`confAUTH_MECHANISMS', `EXTERNAL GSSAPI DIGEST-MD5 CRAM-MD5 LOGIN PLAIN')dnl

……

dnl # The following causes sendmail to only listen on the IPv4 loopback address
dnl # 127.0.0.1 and not on any other network devices. Remove the loopback
dnl # address restriction to accept email from the internet or intranet.
dnl #
DAEMON_OPTIONS(`Port=smtp,Addr=0.0.0.0, Name=MTA')dnl
dnl #
dnl # The following causes sendmail to additionally listen to port 587 for
dnl # mail from MUAs that authenticate. Roaming users who can't reach their
dnl # preferred sendmail daemon due to port 25 being blocked or redirected find
dnl # this useful.
dnl #
dnl DAEMON_OPTIONS(`Port=submission, Name=MSA, M=Ea')dnl
DAEMON_OPTIONS(`Port=587, Name=MSA, M=Ea')dnl

……

dnl #
dnl # The following example makes mail from this host and any additional
dnl # specified domains appear to be sent from mydomain.com
dnl #
dnl MASQUERADE_AS(`mydomain.com')dnl
MASQUERADE_AS(`knowstep.com')dnl

……

dnl MASQUERADE_DOMAIN(localhost)dnl
dnl MASQUERADE_DOMAIN(localhost.localdomain)dnl
dnl MASQUERADE_DOMAIN(mydomainalias.com)dnl
dnl MASQUERADE_DOMAIN(mydomain.lan)dnl
MASQUERADE_DOMAIN(knowstep.com)dnl
MAILER(smtp)dnl
MAILER(procmail)dnl
dnl MAILER(cyrusv2)dnl

```
重新生成sendmail.cf
```shell
 m4 /etc/mail/sendmail.mc > /etc/mail/sendmail.cf
```
2./etc/mail/local-host-names配置
```shell
# local-host-names - include all aliases for your machine here.
knowstep.com
mail.knowstep.com
```
3./etc/mail/submit.cf配置
```shell
……

# my official domain name
# ... define this only if sendmail cannot automatically determine your domain
#Dj$w.Foo.COM
Djknowstep.com

……
```
4./etc/mail/access配置
```shell
# Check the /usr/share/doc/sendmail/README.cf file for a description
# of the format of this file. (search for access_db in that file)
# The /usr/share/doc/sendmail/README.cf is part of the sendmail-doc
# package.
#
# If you want to use AuthInfo with "M:PLAIN LOGIN", make sure to have the 
# cyrus-sasl-plain package installed.
#
# By default we allow relaying from localhost...
Connect:localhost.localdomain		RELAY
Connect:localhost			RELAY
Connect:127.0.0.1			RELAY
Connect:218.85.80.105			RELAY
Connect:192.168.0			RELAY
```
重新生成access.db
```shell
makemap hash access.db < access
```
5./etc/hosts配置
```shell
127.0.0.1 localhost localhost.localdomain knowstep.com
192.168.0.123 game18183-010-cq02bcconline2588-1v4-4-4
#192.168.0.123 knowstep.com
```
6.cyrus-sasl配置
```shell
ln -sf /usr/lib64/sasl2 /usr/local/lib64/sasl2
ln -sf /usr/lib64/sasl2 /usr/local/lib/sasl2
```
在/user/lib64/sasl2中添加2个文件 sendmail.conf、smtpd.conf，内容相同，如下：
```shell
pwcheck_method:saslauthd
saslauthd_path:/var/run/saslauthd/mux
```
测试
```shell
testsaslauthd -u service -p qweasd123
```
7.域名添加指向
A记录：
smtp.knowstep.com  指向服务器IP
mail.knowstep.com  指向服务器IP
MX记录：
指向knowstep.com
8.测试
实现：通过smtp auth方式能在任何IP发送，不通过auth只能在127.0.0.1，218.85.80.105，192.168.0网段发送
PHPMailer包需自己另外下载
auth方式：
```php
<?php
require 'PHPMailerAutoload.php';

$mail = new PHPMailer;

//$mail->SMTPDebug = 3;                               // Enable verbose debug output

$mail->isSMTP();                                      // Set mailer to use SMTP
$mail->Host = '180.76.149.123';  // Specify main and backup SMTP servers
//$mail->Host = 'smtp.knowstep.com';  // Specify main and backup SMTP servers
$mail->SMTPAuth = true;                               // Enable SMTP authentication
$mail->Username = 'service@knowstep.com';                 // SMTP username
$mail->Password = 'qweasd123';                           // SMTP password
//$mail->SMTPSecure = 'ssl';                            // Enable TLS encryption, `ssl` also accepted
$mail->SMTPDebug = 2;
$mail->Port = 587;                                    // TCP port to connect to

$mail->setFrom('service@knowstep.com', 'knowstep');
$mail->addAddress('276887367@qq.com');

$mail->isHTML(true);                                  // Set email format to HTML

$mail->Subject = 'Here is the subject'.time();
$mail->Body    = 'This is the HTML message body <b>in bold!</b>';
$mail->AltBody = 'This is the body in plain text for non-HTML mail clients';

if(!$mail->send()) {
    echo 'Message could not be sent.';
    echo 'Mailer Error: ' . $mail->ErrorInfo;
} else {
    echo 'Message has been sent';
}
```
非auth方式：
```php
<?php
require 'PHPMailerAutoload.php';

$mail = new PHPMailer;

//$mail->SMTPDebug = 3;                               // Enable verbose debug output

$mail->isSMTP();                                      // Set mailer to use SMTP
$mail->Host = '180.76.149.123';  // Specify main and backup SMTP servers
//$mail->Host = 'smtp.knowstep.com';  // Specify main and backup SMTP servers
$mail->SMTPAuth = false;                               // Enable SMTP authentication
$mail->Username = 'service@knowstep.com';                 // SMTP username
$mail->Password = 'qweasd123';                           // SMTP password
//$mail->SMTPSecure = 'ssl';                            // Enable TLS encryption, `ssl` also accepted
$mail->SMTPDebug = 2;
$mail->Port = 25;                                    // TCP port to connect to

$mail->setFrom('service@knowstep.com', 'knowstep');
$mail->addAddress('276887367@qq.com');

$mail->isHTML(true);                                  // Set email format to HTML

$mail->Subject = 'Here is the subject'.time();
$mail->Body    = 'This is the HTML message body <b>in bold!</b>';
$mail->AltBody = 'This is the body in plain text for non-HTML mail clients';

if(!$mail->send()) {
    echo 'Message could not be sent.';
    echo 'Mailer Error: ' . $mail->ErrorInfo;
} else {
    echo 'Message has been sent';
}
```
