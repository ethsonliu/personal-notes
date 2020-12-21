在项目根目录下打开终端，执行，

```bash
hapoa@hapoa-virtual-machine:~/projects/NewBackmanage$ git reflog --date=local | grep p2p
f06cde0 HEAD@{Fri Dec 11 16:35:04 2020}: checkout: moving from feature/p2p to feature/penetrate-udp
4e57295 HEAD@{Fri Dec 11 15:44:21 2020}: checkout: moving from release/v3.22.0 to feature/p2p
1a84653 HEAD@{Thu Dec 10 16:24:22 2020}: checkout: moving from feature/p2p to feature/penetrate-udp
a2d85f0 HEAD@{Thu Dec 10 15:57:55 2020}: checkout: moving from feature/penetrate-udp to feature/p2p
433e977 HEAD@{Wed Oct 28 15:39:40 2020}: checkout: moving from feature/p2p to hotfix/float
a2d85f0 HEAD@{Wed Oct 28 13:44:04 2020}: checkout: moving from release/v3.21.0 to feature/p2p
861ef2e HEAD@{Mon Sep 28 13:36:59 2020}: checkout: moving from feature/p2p to dev
3115bd7 HEAD@{Sun Sep 27 16:05:56 2020}: checkout: moving from release/v3.22.0 to feature/p2p
991fbd2 HEAD@{Sun Sep 27 16:05:06 2020}: checkout: moving from feature/p2p to release/v3.22.0
3115bd7 HEAD@{Sun Sep 27 15:45:19 2020}: checkout: moving from feature/1200 to feature/p2p
23313ab HEAD@{Thu Sep 24 11:47:32 2020}: checkout: moving from feature/p2p to release/v3.21.0
3115bd7 HEAD@{Thu Sep 24 11:47:13 2020}: commit: 完成p2p
861ef2e HEAD@{Tue Sep 22 10:20:47 2020}: checkout: moving from dev to feature/p2p

```

最后一行中可以发现，是从 dev 中拉出来的。
