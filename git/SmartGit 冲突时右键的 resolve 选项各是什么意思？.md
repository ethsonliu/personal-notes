现有一个 exe，如果云端 origin 上已有更新，local 本地也进行了更新，在提交的时候，需要 pull rebase，然后很自然地会出现冲突，此时你就可以右键选择 resolve，
此时弹出四个选项：

![](https://github.com/EthsonLiu/personal-notes/blob/master/_image/028.png)

第二个：直接使用 origin 的那份

第三个：直接使用 local 的那份

## 其它

ours 还是 theirs 在执行不同的操作，含义可能是颠倒的，参见：

- https://nitaym.github.io/ourstheirs/
- https://stackoverflow.com/questions/25576415/what-is-the-precise-meaning-of-ours-and-theirs-in-git

### git merge

let's merge conflicting branch `feature` into `master`

```shell
$ git checkout master
$ git merge feature
Auto-merging Document
CONFLICT (content): Merge conflict in codefile.js
Automatic merge failed; fix conflicts and then commit the result.
```
either fix the conflict manually by editing codefile.js, or use

```shell
$ git checkout --ours codefile.js   # to select the changes done in master
$ git checkout --theirs codefile.js # to select the changes done in feature
```

### git rebase

let's rebase conflicting branch `feature` over `master`

```shell
$ git checkout feature
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: a commit done in branch feature
error: Failed to merge in the changes.
...
```

either fix the conflict manually by editing codefile.js, or use

```shell
$ git checkout --ours codefile.js   # to select the changes done in master
$ git checkout --theirs codefile.js # to select the changes done in feature
```




