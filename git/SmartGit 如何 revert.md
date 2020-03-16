当讨论 revert 时，需要分两种情况，因为 commit 分为两种：

- 一种是常规的 commit，也就是使用 git commit 提交的 commit；
- 另一种是 merge commit，在使用 git merge 合并两个分支之后，你将会得到一个新的 merge commit

merge commit 和普通 commit 的不同之处在于 merge commit 包含两个 parent commit，代表该 merge commit 是从哪两个 commit 合并过来的。

假如有一个 merge commit，使用 git show 命令可以查看 commit 的详细信息，

```
➜  git show bd86846
commit bd868465569400a6b9408050643e5949e8f2b8f5
Merge: ba25a9d 1c7036f
```

这代表该 merge commit 是从 ba25a9d 和 1c7036f 两个 commit 合并过来的。

而常规的 commit 则没有 Merge 行，

```
➜  git show 3e853bd
commit 3e853bdcb2d8ce45be87d4f902c0ff6ad00f240a
```

revert 常规 commit 直接使用 git revert <commit id> 即可，git 会生成一个新的 commit，将指定的 commit 内容从当前分支上撤除。在 SmartGit 上直接右击
你要 revert 的 commit，然后选择 revert 就可以了，很简单。

但是 revert merge commit 有一些不同，在 SmartGit 上直接右击你要 revert 的 merge commit，然后也选择 revert，此时你会发现当前 checkout 的分支处于
reverting 状态，中央列表会出现你要 revert 的文件，右上角会有个 revert abort 终止回撤的按钮。接着全选所有 revert 文件，然后点击上方的 commit，出来弹窗，
选择 commit anyway，即可。

参考：<https://segmentfault.com/a/1190000012897697>
