<p align="center">
  <a href="https://images.guide/">
    <img src='/app/images/logo-banner.jpg' alt="重要的图像优化"/>
  </a>
</p>

<p align="center">
  <a href="https://travis-ci.com/googlechrome/essential-image-optimisation">
    <img alt="Build Status" src="https://travis-ci.com/GoogleChrome/essential-image-optimization.svg?token=x9N5mUsYfWv1zz1h5KfF&branch=master">
  </a>
  <a href="https://www.webpagetest.org/result/170901_EM_348e5bd8fa649e122b4684e0e6febc35/">
    <img alt="WebPageTest" src="https://img.shields.io/badge/webpagetest-report-brightgreen.svg">
  </a>
  <a href="https://www.webpagetest.org/lighthouse.php?test=170901_EM_348e5bd8fa649e122b4684e0e6febc35&run=3">
    <img alt="Lighthouse" src="https://img.shields.io/badge/lighthouse-90+-blue.svg">
  </a>
</p>

## 前提

### [Node.js](https://nodejs.org)

打开终端并输入`node --version`。
Node应该返回0.10.x以上的版本。

如果你未安装Node，请到[nodejs.org](https://nodejs.org)点击那个大个的绿色安装按钮。

### [Gulp](http://gulpjs.com)

打开终端并输入`gulp --version`。
如果Gulp已经安装，应该返回3.9.x以上的版本号。
如果你需要安装或更新Gulp，打开终端并输入：

```sh
$ npm install --global gulp
```

*上面的命令会执行Gulp全局安装。基于你的用户账号，你可能需要在没有管理员权限的情况下全局安装包，请参考如下[系统配置](https://github.com/sindresorhus/guides/blob/master/npm-global-without-sudo.md)方法。*


### 本地依赖

接着，使用命令安装本地依赖项：

```sh
$ npm install
```

### 构建这部书

#### 查看更新和自动刷新设备

```sh
$ gulp serve
```

这条命令可以输出你用于本地测试的IP地址，以及用于其他网络设备连接使用的IP地址。
`serve` 命令运行的服务并不使用[服务工作线程](http://www.html5rocks.com/en/tutorials/service-worker/introduction/)缓存，因此你的站点在Web服务器关闭后也会停止服务。

#### 构建和优化

```sh
$ gulp
```

构建和优化当前项目，并准备部署。

#### 提供完全建设和优化的网站服务

```sh
$ gulp serve:dist
```

####  `serve` 和 `serve:dist`的区别

请一定**注意** `serve` and `serve:dist` 命令任务的不同：

* `serve` 是使用一个无操作的 `service-worker.js` ，它无法离线运行。
* `serve:dist` 则是使用 `workbox` 来生出输出，它可以离线运行。

 `serve` 是运行在3000端口上，而 `serve:dist` 是运行在3001端口上。
如此设计的目的是不同的服务之间不会相互影响。使用 `workbox`生出输出的方式很难快速的进行本地测试，因此不适合用在开发服务环境。

#### 生成这本书的PDF版本

这个项目支持在本地生成PDF版本，安装我们的依赖项，并运行命令：

```sh
$ gulp generate-pdf
```

这个PDF生成是使用了Chrome团队开发的[Puppeteer](https://github.com/GoogleChrome/puppeteer)项目。

如果你觉得这样有点繁琐，并且你只想使用浏览器的“打印到PDF”功能，那么这也是支持的。首先，在[https://images.guide](https://images.guide/)上访问该书，并向下滚动确保所有图像都已经被（延迟）加载，然后就可以按照任何其他网页的方式打印到PDF。

#### 附加项目的一些细节

##### 模板

这里使用另一个非常简单的模板设置。我们将 `app/partials/book.md` 的内容从markdown格式转换到HTML并注入到一个书籍模板 `app/index.html`中。 我使用了 `gulp-md-template` 来实现这一点。

##### 图像

书中的大多数图像都是托管在我的Cloudinary账户上。如果你的合并请求（Pull Request）是想要改进或附加任何图像，大可不必担心，你可以在 `app/images/` 中直接添加它们。我会根据需要将图像托管到Cloudinary并再次提交它们。或者，你可以在合并请求时提醒我，通常我会使用托管到Cloudinary的你的图像的URL恢复到文件中。

##### 语法高亮显示

本书的初始版本采用了一个非常准确的方法来进行语法突出显示。也就是说，使用像[Prism](http://prismjs.com/)这样的轻量级扩展库来更好的突出显示，对该项目来说是一个值得欢迎的贡献。因为，我们希望找到一种不影响页面的核心体验的方式。

#### 关于贡献

我很欢迎你帮助我们改善这本书。如果你有兴趣提出合并请求，请注意：

1. 确保你的请求有个准确的标题和描述。
2. 你的请求只更改需要更改的地方。大多数情况下它会是 `app/partials/book.md`.

如果你更新了本书中的意见或建议，请提供数据以支持所做的更改。这有助于我们对于这些更改做出最好的回应。

##### 关于翻译

如果你有兴趣翻译此书，请提出问题，我们可以交流如何处理。翻译可能会作为这个项目的一部分，或者更好作为一个分支。通过我们之间的协调，我们可以找到一个尽可能让所有版本都同步的方式，以便更好的为读者提供服务。

## 分页

这本书中的内容正在被移植到[Web基础支持](https://developers.google.com/web/fundamentals/)中。我们将会将它作为其中的一个新篇章，叫做自动化图像优化，并且会被分割到几个不同的页面。但同时，我们还是会保持这边文章《重要的图片优化》为单页的格式。

## 开源协议

除非特殊说明，否则本书的内容全部使用[Attribution-NonCommercial-NoDerivs 2.0 Generic (CC BY-NC-ND 2.0)](https://creativecommons.org/licenses/by-nc-nd/2.0/)授权协议，代码示例使用 [Apache 2.0授权协议](http://www.apache.org/licenses/LICENSE-2.0)。版权所有Google, 2017.
