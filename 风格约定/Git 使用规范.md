## 目录

- [issue 标签](#issue-标签)
- [commit message 规范](#commit-message-规范)


## issue 标签

- `bug`：已被确定是一个 bug。如果这个 bug 最终被确定是第三方库的 bug，可以打上`bug(third-party): `，后面紧跟库的名字，若有多个库，以 & 连接，比如，`bug(third-party): lib-a & lib-b`。
- `proposal`：对项目的建议，比如希望增加一个功能。
- `xx-error`：文档有错误，则为`documentation-error`；测试用例有错误，则为`test-error`。
- `typo`：代码或文档有单词拼写错误。
- `support`： 提出的问题需要开发者协作排查，咨询，调试，等日常技术支持。
- `enhancement`：优化相关，如 4 寸屏幕下边框过宽、字过小等等（和`proposal`区别是，`enhancement`的功能已经存在，而`proposal`的功能是目前不存在的）。
- `usage-guide`：在使用上的指导。
- `duplicate`：重复的问题。
- `need-more-information`：问题描述不清。
- `need-reproduce`：需要重现。

## commit message 规范

```
<type>(<scope>): <subject> [<ISSUE_ID>]

<body>

<footer>
```

- type: 是用于说明该 commit 的类型的，一般我们会规定 type 的类型如下，
  - feat: 新功能(feature)
  - fix: 修复 bug
  - docs: 文档(documents)
  - style: 代码格式(不影响代码运行的格式变动，空格、缩进、代码注释的修改等等)
  - refactor: 重构(既不是新增功能，也不是修改 bug 的代码变动)
  - test: 测试
  - revert: 恢复到某一个提交记录
  - chore: 构建或辅助工具的变动
  - misc: 一些未归类或不知道将它归类到什么方面的提交
- scope: 说明 commit 影响的范围，比如数据层，控制层，视图层等等，这个需要视具体场景与项目的不同而灵活变动。
  - 使用第一人称现在时的动词开头，比如 modify 而不是 modified 或 modifies
  - 首字母小写，并且结尾不加句号
- subject: 对于该 commit 目的的简短描述。
- ISSUEE_ID: 假设你的需求或者 bug 修复可能会有对应的 issues 记录，你可以加到你的 commit 信息中如 issue-37938634。
- body: 就是 subject 的详细说明。
- footer: 你可以填写相关的需求管理 issues id
  - 当有非兼容修改(Breaking Change)时必须在这里描述清楚
  - 关联相关 issue，如 Closes #1, Closes #2, #3
  - 如果功能点有新增或修改的，还需要关联文档 doc 和 egg-core 的 PR，如 eggjs/egg-core#123

具体参考：<https://github.com/angular/angular/commits/master>

## 参考

- [egg 代码贡献规范](https://eggjs.org/zh-cn/contributing.html)
- [Git 协同与提交规范](https://www.yuque.com/fe9/basic/nruxq8#6c228def)
- [Angular Commit Message Guidelines](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#commit)
