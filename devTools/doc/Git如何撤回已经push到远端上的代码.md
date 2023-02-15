# Git如何撤回已经push到远端上的代码

1. 首先 `git log`，找到想要撤回的提交的 `commit id`

2. 通过命令 `git reset --soft commit id` 回退代码，回退完成后代码相当于刚写完的状态，即还没有进行add、commit、push的情况

3. 把当前的版本强制提交到远端，`git push origin 分支名 –-force`，一定要加最后的 `--force` 参数，否则会提交失败！
