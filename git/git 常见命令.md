```bash
git status                                                # 查看当前版本状态（是否修改）

git clone ssh://git@example.com/central ~/central

git add .                                                 # 增加当前子目录下所有更改过的文件至index
git commit -m 'xxx'                                       # 提交

git commit -am 'xxx'                                      # 将上述的 add 和 commit 合为一步

git push origin master                                    # 将当前分支 push 到远程 master 分支

git add forgotten_file                                    # 上一次提交有漏的或者还未修改完的文件，修改完后
git commit --amend --no-edit                              # 重新提交，并且上次提交的信息不作修改
git push -f origin FixForBug                              # 最后提交到云端，-f 表示 force，是因为你应该先 pull 然后再 push

git log -n 3                                              # 显示最近的三次提交

git branch                                                # 显示本地分支

git checkout -b master_copy                               # 从当前分支创建新分支 master_copy 并 checkout 到它
git checkout features/performance                         # checkout 到已存在的 features/performance 分支
git checkout -b devel origin/develop                      # 从远程分支 develop 创建新本地分支 devel 并 checkout 到它

git push origin BUG-local:BUG-origin                      # 将本地分支（云端不存在这个分支） BUG-local 推送到云端，分支名为 BUG-origin

git stash                                                 # 暂存当前修改，将所有至为HEAD状态
git stash save 'stashed message'                          # 功能与git stash一样，但message是自己决定的，方便查找
git stash list                                            # 查看所有暂存
git stash apply stash@{0}                                 # 应用第一次暂存
git stash clear                                           # 清除所有暂存

git pull origin master                                    # 获取远程分支 master 并 merge 到当前分支 (option: --rebase)

git branch -d local-branch

git branch -r -d origin/feat1                             # delete remote-tracking branch
git push origin :feat1

```

git cherry-pick: https://ruanyifeng.com/blog/2020/04/git-cherry-pick.html

git fetch: https://www.yiibai.com/git/git_fetch.html
