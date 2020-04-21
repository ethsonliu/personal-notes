## git-flow

假如把一个 feature 合并到 dev，操作如下（两个方法），

- 右键选择 feature，Finish Feature，就可以合并了，注意它会弹窗合并到哪里，合并后是否删除 feature 分支，自己注意选择。
- 先 checkout 到 dev，然后右键 feature，选择 merge。

合并的过程中会出现冲突，SmartGit 默认的冲突解决编辑器分为三个窗口，左边的为 dev ，右边的为 feature，中间的就是你实际要的内容，改完之后关掉窗口，会弹窗是否解决完冲突，解决完就会 stage，点击确定，你就会发现这个文件从冲突的状态变为了 staged。

## dev-master

把 dev 合并到 master，步骤：checkout 到 master，右键 dev 选择 merge，再选择 create-merge-commit，如果没有冲突就会直接合并进去；如果有冲突就解决。
