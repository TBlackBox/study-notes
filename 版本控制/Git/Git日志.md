# 提交日志
不论对于SVN还是Git来说，还有一点尤为重要的就是每次提交到代码库时的日志撰写。很多人都认为日志是很没必要的，浪费时间还没啥用,其实撰写清晰规范的格式化日志有助于追踪版本修改、查看历史记录等。SVN的默认配置是允许提交空日志的，但Git却是不允许日志为空的。现在，市面上有很多类似的提交日志规范，这里推荐使用Angular规范，是目前使用最广的写法，比较合理和系统化，并且有配套的工具。



# Angular规范
Angular规范的commit message包括三个部分：Header、Body 和 Footer，格式如下。
```
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```

说明：  
1. type用于说明 commit 的类别，只允许使用下面7个标识。

+ feat：新功能（feature）
+ fix：修补bug
+ docs：文档（documentation）
+ style： 格式（不影响代码运行的变动）
+ refactor：重构（即不是新增功能，也不是修改bug的代码变动）
+ test：增加测试
+ chore：构建过程或辅助工具的变动

2. scope用于说明 commit 影响的范围，比如数据层、控制层、视图层等等，视项目不同而不同。

3. subject是 commit目的的简短描述，不超过50个字符。

body部分是对本次 commit 的详细描述，可以分成多行。

4. footer部分只用于两种情况: 不兼容变动时，以BREAKING CHANGE开头，后面是对变动的描述、以及变动理由和迁移方法；如果当前 commit 针对某个issue，那么可以在 Footer 部分关闭这个issue。

还有一种特殊情况，如果当前commit用于撤销以前的commit，则必须以revert:开头，后面跟着被撤销 Commit 的 Header。Body部分的格式是固定的，必须写成This reverts commit .，其中的hash是被撤销commit的 SHA 标识符。

如此规范commit message，可以带来以下好处：

1. 提供更多的历史信息，方便快速浏览。可以对代码的历史记录一目了然，能够大体上知道每次提交都做了什么。
```
git log [last tag] HEAD --pretty=format:%s
```

2. 可以过滤某些commit（比如文档改动），便于快速查找信息。比如要查看新增加的功能。
```
git log [last release] HEAD --grep feat

```
3. 使用conventional-changelog可以直接从commit生成change log(发布新版本时，用来说明与上一个版本差异的文档)。
```
conventional-changelog -p angular -i CHANGELOG.md -w
```

此外，可以使用Commitizen来撰写符合规范的commit message。
虽然以上介绍使用Git来做示例的，但是对于SVN也是可以按照此套规范来进行的。
