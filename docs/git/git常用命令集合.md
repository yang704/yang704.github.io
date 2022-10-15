# git 常用命令工具<!-- {docsify-ignore-all} -->

##### git 初始化
```
git init
```

##### git连接远程仓库地址
```
git remote add origin git@gitee.com:mryangzzzzz/test_page.git
```

##### git查看仓库地址
```
git remote -v
```

##### git切换远程仓库地址
```
//方式一
git remote set-url origin git@gitee.com:mryangzzzzz/test_page.git

//方式二:先删除，再添加
git remote rm origin;//删除现有仓库
git remote add origin git@gitee.com:mryangzzzzz/test_page.git;//添加新远程仓库
```

##### git查看远程分支
```
git branch -a
```

##### git切换远程分支
```
git checkout <远程分支名>
```

##### git克隆指定目录
```
git clone  git@gitee.com:mryangzzzzz/test_page.git
```

##### git查看当前项目状态
```
git status
```

##### git添加文件到暂存区
```
git add .
```

##### git将暂存区内容添加到仓库
```
git commit -m '备注'
```

##### git提交代码
```
git push origin master<分支名>
```

##### git回退版本
```
git reset
```

##### git从远程获取代码并合并本地的版本
```
git pull origin<远程主机名> master<远程分支名>：brantest<本地分支名>
```

##### git查看历史提交记录
```
git log
```

##### git生成ssh秘钥
```
git config --global user.name "用户名"

git config --global user.email "邮箱"

//此时在C:\Users\Administrator目录下会生成.gitconfig配置文件，这个文件不能删除;
//接着查看.gitconfig配置文件里的内容
//继续在git命令窗口中输入命令：ssh-keygen -t rsa -C "blkj@boranet.com.cn"，就可以生成SSH公钥和私钥了

ssh-keygen -t rsa -C "自己文件中的地址"
```