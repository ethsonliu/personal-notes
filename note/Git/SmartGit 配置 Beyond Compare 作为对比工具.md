## Diff
依次【Edit】- 【 Preferences】-【Tools】-【Diff Tools】
点击【Add】，填入下面内容，

```
File Pattern: *

选择 External diff tool

# [如果是 Linux 参考： “/usr/bin/bcompare”]
Command: C:\Program Files (x86)\Beyond Compare 4\bcomp.exe

Arguments: /readonly /lefttitle="{leftTitle}" /righttitle="{rightTitle}" "{leftFile}" "{rightFile}"
```

## Merge 
依次【Edit】- 【 Preferences】-【Tools】-【Conflict Solvers】
点击【Add】，填入下面内容，

```
File Pattern: *

选择 External Conflict Solver

Command: C:\Program Files (x86)\Beyond Compare 4\bcomp.exe

Arguments: "{leftFile}" "{rightFile}" "{baseFile}" /mergeoutput="{mergedFile}"
```



参考链接：[https://hackoops.com/180.html](https://hackoops.com/180.html)