# elasticsearch安装配置

1. 添加用户

	``` shell/Users/allan/Documents/gitbook/getting_started/mysql.md

	groupadd elasticsearch
	useradd  -g elasticsearch elasticsearch
	passwd elasticsearch els123456
	chown -R elasticsearch:elasticsearch /fandom/elasticsearch-2.2.0
	```
2. 启动服务

	``` shell
	 bin/elasticsearch -d -p pid
	``` 

3. 安装插件

	``` shell
	plugin install mobz/elasticsearch-head
	plugin install lukas-vlcek/bigdesk
	```
	如果服务器无法连接到外网环境，可以下载插件，然后解压到plugins目录,例如安装head插件：
	在plugins目录下建立[head](git://github.com/mobz/elasticsearch-head.git)和[ik](https://github.com/medcl/elasticsearch-analysis-ik.git)目录，将下载到插件分别拷贝到目录；


