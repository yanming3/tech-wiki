# SSL自签名证书
##1.生成CA根证书
```bash
openssl genrsa -out myCA.key 2048
openssl req -new -X509 -key myCA.key -out myCA.cer -days 36500
```
执行上面的两个操作之后会提示输入以下几个内容,为了显示正常尽量使用英文)：
	•	Country Name (2 letter code) :CN //国家简称
	•	State or Province Name (full name) [Some-State]:Shanghai //州或省的名字
	•	Locality Name (eg, city) :Pudong //区或市县的名称
	•	Organization Name (eg, company) [Internet Widgits Pty Ltd]:shenyun //公司或组织名
	•	Organizational Unit Name (eg, section) :data //单位或者是部门名称
	•	Common Name (e.g. server FQDN or YOUR name) :shenyun //服务器名称或者昵称
	•	Email Address:yanmingyan@gmail.com //Email地址
	
## 2.生成服务端私钥和证书请求
```bash
openssl genrsa -out server.key 2048
openssl req -new -out server.req -key server.key -subj /CN=10.36.40.202
```
CN的内容是服务器的域名或者IP地址
## 3.通过CA签发证书
```bash
openssl x509 -req -in server.req -out server.cer -CAkey myCA.key -CA myCA.cer -days 36500 -CAcreateserial -CAserial serial
```
## 4.生成pem格式证书
```bash
cat server.cer server.key > server.pem
```
## 5.生成pkcs12格式证书
```bash
openssl pkcs12 -export -in server.cer -inkey server.key -out tomcat.p12 -name tomcat -CAfile myCA.cer -caname root -chain
```

## 6.加入到信任证书
```bash
keytool -keystore truststore.jks -keypass 123456 -storepass 123456 -alias ca -import -trustcacerts -file server.pem
```
	


