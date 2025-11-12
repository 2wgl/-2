### 将整个Git代码库迁移到新仓库

**1.克隆旧仓库的镜像**

````shell
git clone --mirror <旧仓库URL>  # 创建裸仓库，包含所有分支、标签和历史
cd <仓库目录>.git  # 进入克隆的裸仓库目录
````

**2.修改远程地址为新仓库**

````shell
git remote set-url origin <新仓库URL>  # 更新远程地址到新仓库
````

**3.推送所有内容到新仓库**

````shell
git push --mirror  # 强制推送所有引用（分支、标签等）
````

#### **迁移后操作**

​		**更新本地远程地址**（若需替换旧仓库）：

````shell
git remote rename origin old-origin  # 可选：重命名旧远程
git remote rename new-origin origin  # 将新远程设置为origin
````

​		**通知团队成员**：

````shell
git remote set-url origin <新仓库URL>   #（让团队成员更新本地仓库地址：）
````

#### PS:生成SSH密钥对

```shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

