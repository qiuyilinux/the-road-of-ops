## 1 安装和配置必要的依赖

```shell
sudo yum install -y curl policycoreutils-python openssh-server perl
sudo systemctl enable sshd
sudo systemctl start sshd
```



## 2 添加GitLab包存储库并安装包

```shell
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
sudo EXTERNAL_URL="https://gitlab.example.com" yum install -y gitlab-ee
```



## 3 重置 root 密码

```shell
1. 进入 git 控制台
 gitlab-rails console
 
2. 建立 root 用户对象
 user = User.where(@root).first
 
3. 修改对象 password 属性
 user.password="12345678"
 
4. 保存
 user.save!
```

