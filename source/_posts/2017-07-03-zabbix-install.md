---
title: Zabbix
date: 2017-07-03 15:08:38
tags: zabbix
---

# 安装

``` bash
cd ~/tmp/

#下载
wget https://jaist.dl.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/3.0.9/zabbix-3.0.9.tar.gz
tar -zxvf zabbix-3.0.9.tar.gz
cd zabbix-3.0.9

#编译安装server和agent
./configure --prefix=/xxx/zabbix --enable-server --enable-agent --with-mysql --with-libcurl
make
make install

#导入数据
cd database;

#导入数据，导入顺序错误，可能会报错
mysql -h{host} -P{port} -u{user} -p{password} {dbname} < mysql/schema.sql
mysql -h{host} -P{port} -u{user} -p{password} {dbname} < mysql/images.sql
mysql -h{host} -P{port} -u{user} -p{password} {dbname} < mysql/data.sql

#复制管理平台到对应目录
#网有第一次访问的时候有安装向导，会检测服务器配置和输入数据库用户名和密码
cd ..
cp -r frontends/php ~/www/zabbix
```

# 修改zabbix server配置
```conf
#日志
LogFile=/xxx/server.log

#数据库相关配置，zabbix无法读取网站的配置，需要单独配置
DBHost=xxx
DBName=xxx
DBUser=xxx
DBPassword=xxx
DBPort=xxx
```

# 修改zabbix agent配置
```conf
LogFile=/xxx/agentd.log
#server IP列表，相当于一个白名单，只有在白名单内的IP才可以读取数据
Server=xxx.xx.xxx.xx,xxx.xx.xxx.xx
#加载子配置，如果要自定义采集点，建议开启，对应的配置文件可以复制到这个目录里
Include=/xxx/zabbix_agentd.conf.d/*.conf
```


# 管理平台配置
* 用apache/nginx配置
* 需要PHP支持
* 有安装向导
* 默认用户名/密码：admin / zabbix

# 一些问题

## 短信配置

### 脚本
本人有短信接口
/xxx/zabbix/share/zabbix/alertscripts/sendsms.php
“/xxx/zabbix” 为zabbix安装目录
``` php
#!/usr/bin/env php
<?php
error_reporting(0);
$to=$argv[1];
$subject=$argv[2];
$body=$argv[3];

$data = [
	'mobile'=>$to,
	'message'=>$subject. '【xxxx】',
];

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, "http://xxxx/v1/send.json");

curl_setopt($ch, CURLOPT_HTTP_VERSION  , CURL_HTTP_VERSION_1_0 );
curl_setopt($ch, CURLOPT_CONNECTTIMEOUT, 30);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
curl_setopt($ch, CURLOPT_HEADER, FALSE);

curl_setopt($ch, CURLOPT_HTTPAUTH , CURLAUTH_BASIC);
curl_setopt($ch, CURLOPT_USERPWD  , 'api:xxxx');

curl_setopt($ch, CURLOPT_POST, TRUE);
curl_setopt($ch, CURLOPT_POSTFIELDS, $data);

$res = curl_exec( $ch );
curl_close( $ch );
$res  = curl_error( $ch );
var_dump($res);
```
脚本需要可执行权限

### 管理平台配置

![管理平台配置](/uploads/zabbix-1.png)

## 邮件配置

### 脚本 
``` php
#!/usr/bin/env php
<?php
$to = $argv[1];
$subject = $argv[2];
$body = $argv[3];

function thislog($info) {
	$fp = fopen('/xxx/zabbix/script.log', 'a');
	fwrite($fp, date('Y-m-d H:i:s'). '  '. $info . PHP_EOL);
	fclose($fp);
}

thislog("start! to:$to subject:$subject body:$body");
try {

	require __DIR__.'/PHPMailer/PHPMailerAutoload.php';

	$mail = new PHPMailer;

	//$mail->SMTPDebug = 3;                               // Enable verbose debug output

	$mail->isSMTP();                                      // Set mailer to use SMTP
	$mail->Host = 'smtp.exmail.qq.com';  // Specify main and backup SMTP servers
	$mail->SMTPAuth = true;                               // Enable SMTP authentication
	$mail->Username = 'xxx';           // SMTP username
	$mail->Password = 'xxx';                        // SMTP password
	$mail->SMTPSecure = 'ssl';                            // Enable TLS encryption, `ssl` also accepted
	$mail->Port = 465;                                    // TCP port to connect to

	$mail->setFrom('xxx', 'Zabbix');
	$mail->addAddress($to, 'bin');

	$mail->Subject = $subject;
	$mail->Body = $body;

	if (!$mail->send()) {

		thislog("send fail! error:$mail->ErrorInfo to:$to subject:$subject body:$body");
		exit(0);
	}
	else {
		thislog("send success! to:$to subject:$subject body:$body");
		exit(1);
	}
} catch (Exception $e) {
    $msg = $e->getMessage();
	thislog("exception: $msg to:$to subject:$subject body:$body");
}
```
依赖 https://github.com/PHPMailer/PHPMailer/

### 管理平台配置
与短信相同




## 报警配置
必须配置：
  * 配置(Administration) -> 动作(Actions)
  * 管理(Administration) -> 用户(Users) -> 报警媒介(Media)

## 自定义监控项
  以后再说吧。。。

## 图表中文乱码解决

上传中文字体到网站：/fonts/目录
本例为微软雅黑字体（下载 uploads/YaHei.Consolas.ttf ）
修改文件 /include/defines.inc.php
``` bash
sed -i 's/DejaVuSans/YaHei.Consolas/g' ~/www/zabbix/include/defines.inc.php
```





