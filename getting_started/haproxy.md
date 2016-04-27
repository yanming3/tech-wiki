# Haproxy安装

## 下载安装文件

```bash
wget http://www.haproxy.org/download/1.6/src/haproxy-1.6.2.tar.gz
```

## 安装
```bash
tar -xvf haproxy-1.6.2.tar.gz
cd haproxy-1.6.2
make TARGET=linux26 ARCH=x86_64 USE_OPENSSL=1
make install
```

## 配置

```bash
mkdir /etc/haproxy
cp examples/option-http_proxy.cfg /etc/haproxy/haproxy.cfg
cp examples/haproxy.init /etc/init.d/haproxy
chmod +x /etc/init.d/haproxy
ln -s /usr/local/sbin/haproxy /usr/sbin/
mkdir /usr/share/haproxy
```

```
global
        maxconn         20000
	ulimit-n	46384
        log             127.0.0.1 local0
        uid             0
        gid             0
        chroot          /usr/local/haproxy-1.6.2
	nbproc		4
        pidfile         /var/run/haproxy.pid
        daemon

defaults
       balance roundrobin
       timeout connect 5000
       timeout client 50000
       timeout server 50000
       option  httplog
       option  httpclose
       

listen admin_stats  
        bind :8888 #监听端口 
        mode http 
        option httplog #采用http日志格式  
        stats refresh 30s #统计页面自动刷新时间  
        stats uri /stats #统计页面url  
        stats realm Haproxy Manager #统计页面密码框上提示文本  
        stats auth admin:fansz123 #统计页面用户名和密码设置  
        #stats hide-version #隐藏统计页面上HAProxy的版本信息 

listen test-service
       bind      :80
       bind      :443 ssl crt /usr/local/haproxy-1.6.2/cert/fansz.pem
       mode      http
       log       127.0.0.1 local0
       option    httplog
       option    httpclose
       server t1 localhost:9080
      # server t1 10.0.0.12:3000 check weight 1 maxconn 1000 check inter 40000
      # server t2 10.0.0.13:3000 check weight 1 maxconn 1000 check inter 40000

listen image
       bind      :8080
       mode      http
       server img1 10.0.0.6:80 check inter 40000

listen upload
       bind      :8090
       mode      http
       server s1 10.0.0.12:2000 check weight 1 maxconn 1000 check inter 40000
       server s2 10.0.0.13:2000 check weight 1 maxconn 1000 check inter 40000

listen codis-console
       bind      :18087
       mode      http
       server s1 10.0.0.2:8080

listen test-codis-console
       bind      :18088
       mode      http
       server s1 10.0.0.12:8080

listen duuo-monitor-console
       bind      :8091
       mode      http
       server s1 10.0.0.6:8080

listen log-console
       bind      :8092
       mode      http
       server s1 10.0.0.7:5601

listen elk-console
       bind      :8093
       mode      http
       server s1 10.0.0.7:9200

```        

## 增加日志记录功能
1. 创建日志文件/var/log/haproxy/haproxy.log

	```bash
	cd /var/log
	sudo mkdir haproxy
	cd haproxy
	sudo touch haproxy.log
	sudo chmod a+w haproxy.log
```

2. 开启rsyslog的日志记录功能

 * 修改/etc/rsyslog.config,去掉**＃ModLoad imudp和＃PServerRun 514**的**＃**
 * 增加haproxy日志配置
 
  ``` bash
 	#Save boot messages also to boot.log
    local7.*           /var/log/boot.log
  ```    
     之后添加
     
    ```bash  
     local0.*             /var/log/haproxy/haproxy.log
    ```
  
* 修改/etc/sysconfig/rsyslog 文件，将

 ```bash
  SYSLOGD_OPTIONS="" 
 ``` 
  修改为
  
  ```bash
  SYSLOGD_OPTIONS="-r -m 0 -c 2"
  ```
  
 3. 修改/etc/haproxy/haproxy.cfg文件，在global区段添加
  
  ```bash
  log         127.0.0.1 local0
  ```

4. 重启rsyslog和haproxy服务，haproxy就能记录日志了

 ```bash
 service rsyslog restart
 service haproxy restart
 ```

5. 监控界面访问地址:
http://121.43.188.107/haproxy?stats


## 配置SSL
## 生成ssl证书

``` shell
		keytool -genkeypair -alias push -keyalg RSA -keysize 2048 -sigalg -validity 365 -keystore push.jks
``` 
## 导出证书和私钥

``` java
        KeyStore ks = KeyStore.getInstance("jks");
        char[] password = "fansz!23456".toCharArray();
        ks.load(new FileInputStream("/Users/allan/Downloads/fansz.jks"), password);

        FileOutputStream kos = new FileOutputStream("/Users/allan/Downloads/key.der");
        Key pvt = ks.getKey("fansz", password);
        kos.write(pvt.getEncoded());
        kos.flush();
        kos.close();
        FileOutputStream cos = new FileOutputStream("/Users/allan/Downloads/cert.der");
        Certificate pub = ks.getCertificate("fansz");
        cos.write(pub.getEncoded());
        cos.flush();
        cos.close();
```        
        
## 生成pem文件

``` shell
			openssl pkcs8 -inform der -nocrypt < key.der > key.pem
			openssl x509 -inform der < cert.der > cert.pem
			cat cert.pem key.pem |tee fansz.pem
```

## 修改haproxy配置

``` shell
		bind      :443 ssl crt /usr/local/haproxy-1.6.2/cert/fansz.pem
		http-request add-header X-Forwarded-Proto https if { ssl_fc }
		redirect scheme https if !{ ssl_fc } ##只允许https
```
## JKS to PKCS12

``` shell
keytool -importkeystore -srckeystore fansz.jks -destkeystore fansz.p12 -srcstoretype JKS -deststoretype PKCS12    
-srcstorepass fanszapp -deststorepass fanszapp -srcalias fansz -destalias   
 fansz -srckeypass fanszapp -destkeypass fanszapp -noprompt
```

## JKS to PEM(ios证书）
``` shell
keytool -importkeystore -srckeystore push.jks -destkeystore push.p12 -srcstoretype JKS -deststoretype PKCS12 -srcstorepass 'push!23456' -deststorepass 'push!23456' -srcalias push -destalias push -srckeypass 'push!23456' -destkeypass 'push!23456' -noprompt


openssl pkcs12 -in push.p12 -out push.pem -passin pass:'push!23456' -passout pass:'push!23456'
```

##TrustedStore

``` shell
keytool -export -alias push -keystore push.jks -rfc -file push.cer
或
keytool -export -alias push -file push.crt -keystore push.jks
keytool -import -alias push -file push.cer  -keystore pushtrusted.jks 
```
## Android证书（bks)
keytool -import -alias push -file push.crt -keystore push.bks -storetype BKS -providerClass org.bouncycastle.jce.provider.BouncyCastleProvider -providerpath bcprov-jdk15on-146.jar


	








