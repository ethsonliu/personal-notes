## 目录

- [issue 标签](#issue-标签)
- [commit message 规范](#commit-message-规范)
- [参考](#参考)

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

参考：

- <https://github.com/microsoft/terminal/issues>

## commit message 规范

```
<type>(<scope>): <subject> [<ISSUE_ID>]

<body>

<footer>
```

每次提交，commit message 都包括三个部分：Header，Body 和 Footer。

### Header

Header 部分只有一行，包括三个字段：type（必需）、scope（可选）和 subject（必需）。

- type: 是用于说明该 commit 的类型的，一般我们会规定 type 的类型如下，
  - feat: 新功能(feature)
  - fix: 修复 bug
  - docs: 文档(documents)
  - style: 代码格式(不影响代码运行的格式变动，空格、缩进、代码注释的修改等等)
  - refactor: 重构(既不是新增功能，也不是修改 bug 的代码变动)
  - test: 增加了测试或者修改了测试
  - revert: 恢复到某一个提交记录
  - perf: 对性能（performance）的改善
  - build: 构建或辅助依赖工具的变动（chore 也可以，但个人推荐用 build）
  - misc: 一些未归类或不知道将它归类到什么方面的提交
- scope 用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。
- subject 是 commit 目的的简短描述，不超过 50 个字符。
  - 以动词开头，使用第一人称现在时，比如 change，而不是 changed 或 changes
  - 第一个字母小写
  - 结尾不加句号（.）

如果 type 为 feat 和 fix，则该 commit 将肯定出现在 Change log 之中。其他情况（docs、chore、style、refactor、test）由你决定，要不要放入 Change log，建议是不要。

### Body

Body 部分是对本次 commit 的详细描述，可以分成多行。下面是一个范例。

```
More detailed explanatory text, if necessary.  Wrap it to 
about 72 characters or so. 

Further paragraphs come after blank lines.

- Bullet points are okay, too
- Use a hanging indent
```

有两个注意点。

（1）使用第一人称现在时，比如使用change而不是changed或changes。

（2）应该说明代码变动的动机，以及与以前行为的对比。

### Footer

Footer 部分只用于两种情况。

（1）不兼容变动

如果当前代码与上一个版本不兼容，则 Footer 部分以 BREAKING CHANGE 开头，后面是对变动的描述、以及变动理由和迁移方法。

```
BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives so there should be no code using it.
```

（2）关闭 Issue

如果当前 commit 针对某个 issue，那么可以在 Footer 部分关闭这个 issue 。

```
Closes #234
```

也可以一次关闭多个 issue 。

```
Closes #123, #245, #992
```

### Revert

还有一种特殊情况，如果当前 commit 用于撤销以前的 commit，则必须以 `revert:` 开头，后面跟着被撤销 Commit 的 Header。

```
revert: "feat(pencil): add 'graphiteWidth' option"

This reverts commit 667ecc1654a317a13331b17617d973392f415f02.
```

Body 部分的格式是固定的，必须写成 `This reverts commit $hash>.`，其中的 hash 是被撤销 commit 的 SHA 标识符。

如果当前 commit 与被撤销的 commit，在同一个发布（release）里面，那么它们都不会出现在 Change log 里面。如果两者在不同的发布，那么当前 commit，会出现在 Change log 的 Reverts 小标题下面。

具体参考：

- <https://github.com/angular/angular/commit/ad987021ce5a2a2bc498dcb38a8e64300c461cc9>
- <https://github.com/angular/angular/commit/85b551a38829f90d4b87cd2a6fa506dfdeed2ec9>

## 参考

- <https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html>
- [egg 代码贡献规范](https://eggjs.org/zh-cn/contributing.html)
- [Git 协同与提交规范](https://www.yuque.com/fe9/basic/nruxq8#6c228def)
- [Angular Commit Message Guidelines](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#commit)
