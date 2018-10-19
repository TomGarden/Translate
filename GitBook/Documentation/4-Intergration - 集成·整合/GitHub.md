## 0x01、 GitHub

使你 GitBook 内容和 GitHub repositry 保持同步。

GitBook 为与 GitHub 双向集成提供支持：在 GitBook 编辑的内容可以被提交到 GitHub repository ，并且 GitHub repository 会被导入一个 commites 推送。

## 0x02、 步骤
和 GitHub 集成非常容易，只需要简单的几部：

1. 在你的主页点击 integration(集成) 按钮，在展开的列表中剑姬 GitHub。
2. 为 GitBook 取得访问你 GitHub 账户权限进行 **认证/授权** 。
3. 选择一个你希望进行关联的 repository 。

一旦你完成上述操作， GitBook 将像你的 repository 中添加一个 web hook ，这使你可以从 repository 中获得每次更新的内容。
当使用 GitBook 变更内容的时候，一个新的 commit 将被 push 。

就是说！你可以从 GitBook 和 GitHub 编辑你的项目了。

### 2.1、 关联一个空 repository
当 GitHub 中的目标 repository 是空的时候，GitBook 将会般当前项目的当前内容作为进行一次初始化提交。

### 2.2、 关联一个非空 repository
当 GitHub 中的目标 repository 是非空的时候，你将被提示选择一个进行使用(GitBook 或者 GitHub)。
- 当选择 GitHub ，repository 中的内容将会替换你当前的 GitBoook 空间中的内容。
- 当选择 GitBook ，你当前 GitBook 空间中的内容会被 push 到 repository。

## 0x03、 Branches(分支)
默认的，GitBook 仅支持 `master` branch。
在设置的时候，可以也选择其他 branch 。
在所选定的 branch 中的每一次新的 commit 都会被作为一个特定的 GitBook 版本导入:
- ![2018-09-10-Link_your_GitHub_repository.png](/GitBook/Image/2018-09-10-Link_your_GitHub_repository.png)

你可以使用模式匹配 branch。
例如当输入 `master doc/*` ,`master` 以及该分支下所有 `doc/` 路径下的内容将会被作为一个版本导入。


## 0x04、 内容配置
为 GitHub 同步指定配置信息可以在 `.gitbook.yaml` 文件中进行例如：
```.gitbook.yaml
root: ./
​
structure:
  readme: README.md
  summary: SUMMARY.md
​
redirects:
  previous/page: new-folder/page.md
```

### 4.1、 root
设置在你文档中寻找的路径，默认为 repository 的根路径。
这一句的含义是告诉 GitBook 在 `docs` 文件夹下进行寻找：
```.gitbook.yaml
root: ./docs/
```

**所有其他选项所指定的 path 是以 root 文件夹作为为基准指定的相对 path**。
所以如果你定义的 root 是 `./docs/` 并且 `structure.sumary` 为 `./product/SUMMARY.md` ，GitBook 将会在 `./docs/product/SUMMARY.md` 文件夹进行实际的文件查询。

### 4.2、 structure 结构
接受两个属性：
- **readme**: 你文档的首页，默认是 `./README.md`
- **summary**: 你文档的目录，默认是 `./README.md`
    - *这一句话原文该是有误吧*

这两个属性的值都是 path ，指向响应的文件。
path 是相对于 "root" 选项的。
例如，当前你的 GitBook 在 `./product` 文件夹下进行文件查找，那么首页的概述如下：
```.gitbook.yaml
structure:
  readme: ./product/README.md
  summary: ./product/SUMMARY.md
```

#### 4.2.1、 summary 概述
`summary` 文件是一个 Markdown 文件(`.md`),他应该有如下的结构
```
# Summary
​
## Use headings to create page groups like this one
​
* [First page's title](page1/README.md)
    * [Some child page](page1/page1-1.md)
    * [Some other child page](part1/page1-2.md)
* [Second page's title](page2/README.md)
    * [Some child page](page2/page2-1.md)
    * [Some other child page](part2/page2-2.md)
    
## A second page group
​
* [Yet another page](another-page.md)
```

提供一个概述选项。
如果你没有指定一个 summary，并且，GitBoook 没有在你文档的根目录发现 `SUMMARY.md` 文件，GitBook 将会根据文件夹结构和后续的 Markdown 文件推断一个包含内容的表结构。

### 4.3、 redirects 重定向
你可以通过在指定到响应文件的方式创建自定义重定向，使一个 URL 指向一个 page 。
path 是相对于 "root" 选项的。
例如，你可以通过重定向使用户访问 GitBook `/help` 页面：
```.gitbook.yaml
redirects:
  help: ./support.md
```

注意，URL 禁止以斜杠开头。 
下面是如何处理更复杂的 URL ：
```.gitbook.yaml
redirects:
  help/contact: ./contact.md
```

> 使用 Git ，移动一个文件的结果是当前文件被移除，一个新的文件被创建。
  这使 GitBook 无法得知文件夹已经被重命名。
  添加重定向之前一定要再三检查。

## 0x05、 FAQ
### 5.1、 GitBook 没有使用 `docs` 文件夹？
默认的，GitBook 使用 repository 的根路径作为起始点。
可以指定一个文件夹作为 markdown 文件的范围。

### 5.2、 GitBook 正在创建一个新的 Markdonw 文件？
在一个已经存在的 repository **当同步操作正在进行的时候在 GitBook 进行文本编辑**，GitBook 可能会创建一个新的 markdown 文件取代原的文件。

### 5.3、 我的 repository 没有列出来？
如果你没有看到你在 GitHub 可以访问的 repositories ，像你所处的组织的 repositories ，可一在 GitHub 查看 [GitBook OAuth application settings page](https://github.com/settings/connections/applications/c9c25e33d347c9b960e3)
只需要简单的点击你将会被授予(或请求) GitBook OAuth app 访问你的 repositories 。
- ![2018-09-10_GitBook_OAuth_application_settings_page.png](/GitBook/Image/2018-09-10_GitBook_OAuth_application_settings_page.png)

## 0x06、 Limitations 限制
### 6.1、 HTML
Markdown 不支持任意 HTML 内容。
GitBook 将会解析其中的一些，但是更多的将被忽略。
你可以不顾及这些依旧使用 HTML ，但是要知道 GitHub 会忽略它们。

### 6.2、 Images
 GitBook 极大程度的简化了图像处理。
图片总是居中的块，它的大小由浏览器的视图窗口大小决定。

这里有几个事：
- **内嵌图像不被支持**取而代之的是图像块
- 你应该首先使用具有适当自然尺寸的图像。

### 6.4、 Plugins 插件
以下插件的语法依然被支持并且他们已经成为 GitBook 的一流功能：

- youtube
- hint
- theme-api

但是插件并非全局支持。
从 [important differences](https://docs.gitbook.com/v2-changes/important-differences#plugins)读取关于插件的更多内容。

### 6.5、 Git Tags
GitHub集成将Git分支同步为版本，但不支持同步Git标记。 解决方法是仍将标记跟踪为分支：
```
git checkout -b v1.0 v1.0
```