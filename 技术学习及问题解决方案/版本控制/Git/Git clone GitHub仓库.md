## 使用ssh克隆GitHub仓库

```bash
# macOS 13版本以下可以使用以下方法进行配置
git config --global user.name "baijiahao02"
git config --global user.email "baijiahao02@58.com"
ssh-keygen -t rsa -C "baijiahao02@58.com" -b 4096
cat ~/.ssh/id_rsa.pub  # 将此内容配置到自己igit中的SSH密钥中
ssh -T igit.58corp.com # 测试链接情况
----------------------------------------------------------
# macOS 13及以上版本需使用以下方法进行配置
git config --global user.name "baijiahao02"
git config --global user.email "baijiahao02@58.com"
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub # 将此内容配置到自己igit中的SSH密钥中
ssh -T igit.58corp.com # 测试链接情况

# ~/.ssh/config 文件需包含一下行
Host *
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_ed25519
```

