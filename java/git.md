#### git命令

##### 1. git列出所有的文件

>  git ls-files

##### 2. git不再跟踪fileName, 但是fileName文件仍保留

> git rm --cached fileName

##### 3. git不再跟踪fileName, 同时fileName文件被删除

> git rm -f fileName

##### 4. git pull 强制覆盖本地的代码

```
git fetch --all
git reset --hard origin/master
git reset --hard origin/<branch_name>
```

```
git fetch apache
git rebase apache/master
git push origin <branch_name> --force 
```

##### 5. git的分支重命名

> git branch -m bugfix/issue-namespace  bugfix/6775

##### 6. git设置代理和取消代理

``` 
git config --global http.proxy http://172.17.10.100:8118
git config --global https.proxy http://172.17.10.100:8118/
```

``` 
git config --global --unset http.proxy
git config --global --unset https.proxy
```

##### 7. git显示remote信息

> git remote -v

##### 8. git切换远程分支

> git checkout -b max-retry-count origin/feature/max-retry-count

##### 9. git显示历史提交信息

> git log

##### 10. 强制提交

``` 
git push origin qa --force --allgit push origin qa --force --tags
```