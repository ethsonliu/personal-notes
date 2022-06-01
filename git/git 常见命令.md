```bash
git status                                                # 查看当前版本状态（是否修改）

git add .                                                 # 增加当前子目录下所有更改过的文件至index
git commit -m 'xxx'                                       # 提交

git commit -am 'xxx'                                      # 将上述的 add 和 commit 合为一步

git log -n 3                                              # 显示最近的三次提交

git branch                                                # 显示本地分支

git checkout -b master_copy                               # 从当前分支创建新分支 master_copy 并 checkout 到它
git checkout features/performance                         # checkout 到已存在的 features/performance 分支
git checkout -b devel origin/develop                      # 从远程分支 develop 创建新本地分支 devel 并 checkout 到它

git push origin BUG-local:BUG-origin                      # 将本地分支（云端不存在这个分支） BUG-local 推送到云端，命令为 BUG-origin

git stash                                                 # 暂存当前修改，将所有至为HEAD状态
git stash list                                            # 查看所有暂存
git stash apply stash@{0}                                 # 应用第一次暂存
git stash clear                                           # 清除所有暂存
```
