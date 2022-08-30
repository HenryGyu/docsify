# 基于Tag快速修复线上BUG

- 假设你是在本地dev分支开发，先将本地未完成的任务暂存

```shell
$ git stash
```

- 基于指定tag版本创建一个分支

```shell
$ git checkout -b fix-branch api-v4.3.69
```

- 执行增删改文件...

```shell
$ git add .
# (此处省略各种操作...)
$ git commit -m “紧急修复说明”
```

- 将本地最新代码发布成tag版本

```shell
$ git tag api-v4.3.69-fix
```

- 将本地tag版本推送到远程

```shell
git push origin api-v4.3.69-fix
```

- 利用Jenkins对刚刚的tag进行打包发布

```shell
# (此处省略各种操作...)
```

- 删除本地tag

```shell
git tag -d api-v4.3.69-fix
```

- 删除远程tag

```shell
git push origin :refs/tags/api-v4.3.69-fix
```

- 切换回dev分支

```shell
git checkout dev
```

- 合并本地的临时修复分支到当前分支（dev）

```shell
git merge fix-branch
```

- 删除本地的临时修复分支

```shell
git branch -d fix-branch
```

- 恢复最近的缓存到当前文件中，同时删除恢复的缓存条目。继续开发

```shell
git stash pop
```
