假如把一个 feature 合并到 dev，操作如下，

右键选择 feature，Finish Feature，就可以合并了。

合并的过程中会出现冲突，SmartGit 默认的冲突解决编辑器分为三个窗口，左边的为 dev ，右边的为 feature，中间的就是你实际要的内容，改完之后关掉窗口，会弹窗是否解决完冲突，解决完就会 stage，点击确定，你就会发现这个文件从冲突的状态变为了 staged。

