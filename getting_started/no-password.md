# 免密钥登录

1. 客户端创建密钥

```bash
ssh-keygen -t rsa
```

2. 将客户端密钥负责到服务端的用户目录下
3. 对于非root用户，需要调整目录权限

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

