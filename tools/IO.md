#磁盘IO性能测试
Linux下可以采用dd测试磁盘的读写性能；
## 测试写速度
```bash
time dd if=/dev/zero of=/tmp/test bs=8k count=1000000
```

## 测试读速度
```bash
time dd if=/tmp/test of=/dev/null bs=8k
```

## 测试读写速度
```bash
time dd if=/tmp/test of=/var/test bs=64k
```

## 参数说明
1. time 有计时作用，dd用于复制，从if读出，写到of;

2. if=/dev/zero不产生IO,因此可以用来测试纯写速度
3. of=/dev/null不产生IO,因此可以用来测试纯读速度
4. 将/tmp/test拷贝到/var/则同时测试了读写速度；
5. bs是每次读写的大小，即一个块的大小，count是读写块的数量
## 测试结果样例
![测试结果](/Users/allan/Documents/gitbook/resources/dd.png)


