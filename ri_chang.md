# 日常

##1. 服务器强制覆盖本地git

```git
git fetch --all  
git reset --hard origin/master
```

##2 `push`代码时只提示输入密码

```git
404-K:*** maizhikun$ git push origin master
git@***.net's password:
```

拉代码时,只提示要输入密码,其实这个是要ssh进入服务器的密码,因为代码的ssh没配置好

##3. `fatal: protocol error: bad line length character: No s`

因为ssh在项目中配置了只读.

```git
404-K:*** maizhikun$ git push origin master
fatal: protocol error: bad line length character: No s
```

配置好ssh就行