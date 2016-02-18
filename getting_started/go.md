# GO
1. 安装包下载

	```shell
	wget https://storage.googleapis.com/golang/go1.5.3.linux-amd64.tar.gz
	tar -xvf go1.5.3.linux-amd64.tar.gz -C /usr/local/
	```
2. 修改环境变量
在目录/etc/profile.d中增加go.sh文件，内容如下：
   
	```shell
	export GOROOT=/usr/local/go
	export GOPATH=/usr/local/codis
	export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
	```
	执行命令```source /etc/profile```使环境变量生效
	


