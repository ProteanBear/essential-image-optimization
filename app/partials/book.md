### <a id="the-tldr" href="#the-tldr">概要</a>

**我们都应该让图片压缩自动化！**

在2017年，我们应该开始让图片优化自动化了。现在，对于Web来说，一个很容易被忽略的实践准则变化就是：一个不经过构建发布流程的内容是很容易出问题的。那么图片优化如何自动化，可以在构建过程中使用[imagemin](https://github.com/imagemin/imagemin)或者[libvips](https://github.com/jcupitt/libvips)，另外还有很多解决方案。

大多数的CDN（如[Akamai](https://www.akamai.com/us/en/solutions/why-akamai/image-management.jsp)）和第三方解决方案如[Cloudinary](https://cloudinary.com/)、[imgix](https://imgix.com/)、[Fastly's Image Optimizer](https://www.fastly.com/io/)、[Instart Logic's SmartVision](https://www.instartlogic.com/technology/machine-learning/smartvision)或者[ImageOptim API](https://imageoptim.com/api)都提供了全面的自动化图像优化方案。

实际上，如果你自己处理，那么阅读博客文章和调整配置的时间成本可能远远大于这些服务的每月费用（Cloudinary是有免费版本的）。如果你基于成本或延时问题，不想将此工作外包给第三方，那么依然有开源项目可以替代，比如[Imageflow](https://github.com/imazen/imageflow)或[Thumbor](https://github.com/thumbor/thumbor)都可以搭建自己的图片优化自动化服务。

> 译者：中国国内类似的CDN，如[七牛云](https://www.qiniu.com/)。



**每个人都应该有效地压缩图片！**


至少，要使用[ImageOptim](https://imageoptim.com/)。它可以显著的减少图片的大小，同时保持图片的显示质量，并且支持Windows和Linux。

更进一步的话，可以使用[MozJPEG](https://github.com/mozilla/mozjpeg)来处理你的JPEG图片（最好设置质量为80%或者更低）并考虑支持渐进式的JPEG；使用[pngquant](https://pngquant.org/)处理PNG图片；使用[SVGO](https://github.com/svg/svgo)处理SVG。通过这些组件可以明确的删除图片文件的元数据，使得图片不会过于庞大（pngquant中使用--strip选项）。不要使用体积疯狂的GIF动画，取而代之提供[H.264](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC)动画（或者为Chrome，Firefox和Opera浏览器提供[WebM](https://www.webmproject.org/)）！如果不行，至少使用[Giflossy](https://github.com/pornel/giflossy)对GIF进行优化。如果你有更强劲的CPU，需要高质量的网络图片，并且接受比较缓慢的编码速度，那么你可以尝试使用：[Guetzli](https://research.googleblog.com/2017/03/announcing-guetzli-new-open-source-jpeg.html)。

一些浏览器可以通过HTTP请求的Header设置支持的图片格式。这个可以用于有条件的提供图片格式：比如为基于Blink的浏览器（Chrome）提供[WebP](https://developers.google.com/speed/webp/)，而为其他浏览器返回JPEG/PNG格式。

事实上，你总是可以做到更多。一些工具可以生成和提供srcset属性中的断点。在基于Blink的浏览器中，你可以通过[client-hints](https://developers.google.com/web/updates/2015/09/automating-resource-selection-with-client-hints)进行自动化处理资源格式的选择，并且你可以发送较少的字节给在浏览器中选择了“数据节省”的用户（通过Header中的[Save-Data](https://developers.google.com/web/updates/2016/02/save-data)）。

图片的大小越小，你为你的用户提供的网络体验就越好，尤其对于移动设备来说。在这本书中，我们将研究通过现代压缩技术减少图像大小并且对质量影响最小的方法。

<details>
<summary><h2>目录</h2></summary>
<p>
<ul>
        <li><a href="#introduction">介绍</a></li>
        <li><a href="#do-my-images-need-optimization">如何判断我的图像是否需要优化？</a></li>
        <li><a href="#choosing-an-image-format">如何选择正确的图像格式？</a></li>
        <li><a href="#the-humble-jpeg">“素人”JPEG</a></li>
        <li><a href="#jpeg-compression-modes">JPEG的压缩模式</a>
                <ul>
                        <li><a href="#the-advantages-of-progressive-jpegs">渐进式JPEG的优点</a></li>
                        <li><a href="#whos-using-progressive-jpegs-in-production">谁在生产环境中使用了渐进式JPEG？</a></li>
                        <li><a href="#the-disadvantages-of-progressive-jpegs">渐进式JPEG的缺点</a></li>
                        <li><a href="#how-to-create-progressive-jpegs">如何生成一个渐进式JPEG？</a></li>
                        <li><a href="#chroma-subsampling">色度（或颜色）抽样</a></li>
                        <li><a href="#how-far-have-we-come-from-the-jpeg">JPEG引发的格式拓展</a></li>
                        <li><a href="#optimizing-jpeg-encoders">使用JPEG优化编码器</a></li>
                        <li><a href="#what-is-mozjpeg">什么是MozJPEG？</a></li>
                        <li><a href="#what-is-guetzli">什么是Guetzli？</a></li>
                        <li><a href="#mozjpeg-vs-guetzli">MozJPEG与Guetzli孰优孰劣？</a></li>
                        <li><a href="#butteraugli">Butteraugli</a></li>
                </ul>
        </li>
        <li><a href="#what-is-webp">什么是WebP？</a>
                <ul>
                        <li><a href="#how-does-webp-perform">WebP的表现如何？</a></li>
                        <li><a href="#whos-using-webp-in-production">谁在生产环境中使用WebP？</a></li>
                        <li><a href="#how-does-webp-encoding-work">WebP编码如何执行？</a></li>
                        <li><a href="#webp-browser-support">WebP的浏览器支持</a></li>
                        <li><a href="#how-do-i-convert-to-webp">如何将我的图像转换为WebP？</a></li>
                        <li><a href="#how-do-i-view-webp-on-my-os">如何在我的操作系统上查看WebP图像？</a></li>
                        <li><a href="#how-do-i-serve-webp">如何提供WebP？</a></li>
                </ul>
        </li>
        <li><a href="#svg-optimization">SVG优化</a></li>
        <li><a href="#avoid-recompressing-images-lossy-codecs">避免使用有损编解码器重复压缩图像</a></li>
        <li><a href="#reduce-unnecessary-image-decode-costs">减少不必要的图像解码和图像调整的开销</a>
                <ul>
                        <li><a href="#delivering-hidpi-with-srcset">对HiDPI图像使用<code>srcset</code></a></li>
                        <li><a href="#art-direction">艺术性方向</a></li>
                </ul>
        </li>
        <li><a href="#color-management">颜色管理</a></li>
        <li><a href="#image-sprites">图像合并</a></li>
        <li><a href="#lazy-load-non-critical-images">延迟加载非关键图像</a></li>
        <li><a href="#display-none-trap">避免<code>display: none;</code>的陷阱</a></li>
        <li><a href="#image-processing-cdns">图像CDN对你有意义吗？</a></li>
        <li><a href="#caching-image-assets">缓存图像资源</a></li>
        <li><a href="#preload-critical-image-assets">预加载关键图像</a></li>
        <li><a href="#performance-budgets">图像性能预估</a></li>
        <li><a href="#closing-recommendations">最终建议</a></li>
        <li><a href="#trivia">备注</a></li>
</ul>
</p>
</details>

### <a id="introduction" href="#introduction">介绍</a>

**图片依然是加载缓慢的首要原因

图片经常占用大量的互联网带宽，因为它们通常都非常大。根据[HTTP Archive](http://httparchive.org/)的统计结果显示，网页数据中平均60%的内容都是有JPEG、PNG和GIF组成的图像。截止至2017年7月，平均一个网站的内容数据有3.0MB，而其中1.7MB是图片。

根据[Tammy Everts](https://calendar.perfplanet.com/2014/images-are-king-an-image-optimization-checklist-for-everyone-in-your-organization/)的研究表明，将图片添加到网页中或者将现有的图片扩大，已经被证明可以提高网站的用户转化率。图片是不可能消失的，所以花费成本在有效的压缩策略上以最小化这种带宽占用将变得非常重要。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image00.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image00.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image00.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image00.jpg"
        alt="Fewer images per page create more conversions. 19 images per page on average converted better than 31 images per page on average." />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image00.jpg"/>
</noscript>
</picture>
<figcaption>根据[Soasta/Google research](https://www.thinkwithgoogle.com/marketing-resources/experience-design/mobile-page-speed-load-time/)2016年的统计，图像是第二高的用户转换因子，其中转换率最好的页面中图片占比38%一下。</figcaption>
</figure>

图片优化包括了可以减少图片文件大小的多种不同方法。它最终取决于您的图片需要什么视觉保真度。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/image-optimisation"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/image-optimisation"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/image-optimisation" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/image-optimisation"
        alt="Image optimization covers a number of different techniques" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/image-optimisation"/>
</noscript>
</picture>
<figcaption><strong>图像优化:</strong> 选择正确的存储格式、有效的压缩，并将关键图片优先于可以延迟加载的其他图片进行加载。</figcaption>
</figure>

通常的图片优化包括了压缩，并且支持通过[`<picture>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture)/[`<img srcset>`](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)标签响应式的压缩它们，并且调整图片的尺寸以降低解码消耗。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502834117/chart_naedwl.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502834117/chart_naedwl.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502834117/chart_naedwl.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502834117/chart_naedwl.jpg"
        alt="A histogram of potential image savings from the HTTP Archive validating the 30KB of potential image savings at the 95th percentile." />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502834117/chart_naedwl.jpg"/>
</noscript>
</picture>
<figcaption>根据[HTTP Archive](http://jsfiddle.net/rviscomi/rzneberp/embedded/result/)统计, CDF (累积分布函数)为95%的每个图像可以节省体积达到30KB！</strong></figcaption>
</figure>

说明我们有足够的空间来统一优化我们的图片。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502519576/essential-image-optimization/image-optim.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502519576/essential-image-optimization/image-optim.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502519576/essential-image-optimization/image-optim.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502519576/essential-image-optimization/image-optim.jpg"
        alt="ImageOptim in use on Mac with a number of images that have been compressed with savings over 50%" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502519576/essential-image-optimization/image-optim.jpg"/>
</noscript>
</picture>

<figcaption>ImageOptim是一个免费的工具，可以通过现代压缩技术和去除不必要的EXIF元数据的方法减少图像文件体积。
</figcaption>
</figure>

如果你是个设计师，那么还有一个用于Sketch的[ImageOptim插件](https://github.com/ImageOptim/Sketch-plugin)，它在导出资源时会自动优化。你会发现它能最大化的节省你的时间。

### <a id="do-my-images-need-optimization" href="#do-my-images-need-optimization">如何判断我的图像是否需要优化？</a>

可以通过[WebPageTest.org](https://www.webpagetest.org/)网站进行图片检查，它将指导你如何更好的优化图片（注意“Compress Images”的内容，即压缩图片）。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image1.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image1.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image1.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image1.jpg"
        alt="WebPage test supports auditing for image compression via the compress images section" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image1.jpg"/>
</noscript>
</picture>

<figcaption>WebPageTest报告中的 "Compress Images" 部分列出了如何更有效地压缩的图像，以及能节省的文件大小。 
</figcaption>
</figure>


<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image2.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image2.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image2.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image2.jpg"
        alt="image compression recommendations from webpagetest" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image2.jpg"/>
</noscript>
</picture>
</figure>


[在Chrome中，[Lighthouse](https://github.com/GoogleChrome/lighthouse)工具可以有效的帮助你的网站达到最佳体验。它包括了图像优化的审核，并且对是否可以进一步压缩图片提出建议，或者指出哪些是屏幕外的图片可以被延迟加载。

而对于Chrome60以上的版本，Lighthouse已经默认被嵌入到Chrome DevTools中的Audits面板中。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/hbo.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/hbo.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/hbo.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/hbo.jpg"
        alt="Lighthouse audit for HBO.com, displaying image optimisation recommendations" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/hbo.jpg"/>
</noscript>
</picture>
<figcaption>Lighthouse可以审核Web性能、找到最佳实践，并测试渐进式Web应用程序的功能。</figcaption>
</figure>


你也可以去了解一下其他的性能审核工具，比如[PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)或者Cloudinary的[Website Speed Test](https://webspeedtest.cloudinary.com/)，这些工具都有丰富的图片优化审核功能。

## <a id="choosing-an-image-format" href="#choosing-an-image-format">如何选择正确的图像格式？</a>

正如Ilya Grigorik在他出色的《[图像优化指南](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/image-optimization)》中指出的，所谓的“正确格式”一定是视觉效果和功能需求的统一。你是使用Raster（位图）还是Vector（矢量图）？

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502469573/essential-image-optimization/rastervvector.png"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502469573/essential-image-optimization/rastervvector.png"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502469573/essential-image-optimization/rastervvector.png" />

<img
        class="lazyload very-small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502469573/essential-image-optimization/rastervvector.png"
        alt="vector vs raster images"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502469573/essential-image-optimization/rastervvector.png"/>
</noscript>
</picture>
</figure>


[[Raster Graphics（位图）](https://en.wikipedia.org/wiki/Raster_graphics):通过对图像的矩形网格内的每个像素的值进行编码来表示图像。它们在缩放时会失真。WebP格式或者应用更广泛的格式比如JPEG或PNG都可以处理为位图，这种格式被应用于要求真实感的地方。Guetzli，MozJPEG和我们讨论的其他组件都适用于优化位图。

[Vector graphics（矢量图）](https://en.wikipedia.org/wiki/Vector_graphics)：使用点、线和多边形来表示图像，并被应用于使用简单几何形状表示的图像（例如徽标）。这种格式可以在高分辨率下显示，并且缩放时不会失真。像SVG就是处理矢量图的好选择。

选择了错误的格式可能会浪费你很多的宝贵时间。但选择正确格式的过程可能一波三折，所以消耗一些时间去尝试其他保存格式对于最终选择正确的格式来说也是值得的。

杰里米·瓦格纳（Jeremy Wagner）在描述他的图像优化理论时，已经对图像格式的[权衡评估](http://jlwagner.net/talks/these-images/#/2/2)做出了讨论。

## <a id="the-humble-jpeg" href="#the-humble-jpeg">“素人”JPEG</a>

[JPEG](https://en.wikipedia.org/wiki/JPEG)可能是世界上使用最广泛的图像格式。如前所述，HTTP Archive抓取的站点上看到的图像中有[45％](http://httparchive.org/interesting.php)是JPEG格式。您的手机、数码相机、旧的网络摄像机——一切的设备都支持这种编解码器。它也很确实很古老，第一次发布可以一直追溯到1992年。在这期间，已经有很多人进行了大量研究，试图改进它，让它变得更好。

JPEG是一种有损压缩算法，它丢弃信息以节省存储空间，并在尝试保持文件尽可能小的同时保留图像质量的方面做出了许多努力。

##### 你的用例可以接受什么样的图像质量？

像JPEG这样的格式最适用于那些具有多个颜色区域的照片或图像。而且大多数优化工具都允许您设置您喜欢的压缩级别；较高级别的压缩可以减小文件的大小，但可能会引起重影、光晕或马赛克等失真效果。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image5.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image5.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image5.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image5.jpg"
        alt="JPEG compression artifacts can be increasingly perceived as we shift from best quality to lowest" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image5.jpg"/>
</noscript>
</picture>
<figcaption>JPEG: 从最佳质量转变到最低质量时，可以感觉到的压缩影响可能会增加。请注意，不同工具中的质量得分可能会有较大的差异。</figcaption>
</figure>

选择什么样的质量设置，请根据你的业务需求：

- **最高质量**：在质量要求远比带宽重要时使用。可能是因为这个图片在你的设计中很重要或者它需要全分辨率显示。
- **一般质量**：当你追求较小的图片，但又不想对图片质量产生较大影响时使用。你的用户可能仍然希望看到的图片比较清晰。
- **较低质量**：当网络带宽更重要时使用。这些图片更适用于不稳定或带宽较低的网络。
- **最低质量**：节省带宽是至关重要时使用。用户可以接受一个较差的视觉体验，以便更快速的加载页面。

下面，我们来谈谈JPEG的压缩模式，因为它们对图片的视觉体验有很大的影响。

<aside class="note"><b>注意：</b> 我们有时有可能会高估用户需要的图像质量。图像质量可能被认为是高保真资源的一个偏差值。但它同样也可以是很主观的。</aside>

## <a id="jpeg-compression-modes" href="#jpeg-compression-modes">JPEG的压缩模式</a>

JPEG图像格式具有多种不同的[压缩模式](http://cs.haifa.ac.il/~nimrod/Compression/JPEG/J5mods2007.pdf)。其中，三种流行的模式分别是基线（顺序），渐进式JPEG（PJPEG）和无损。

##### 基线（或顺序）JPEG和渐进式JPEG有什么不同？

基线JPEG（大多数图像编辑和优化工具的默认项）以相对简单的方式进行编码和解码：从上到下。当基线JPEG加载缓慢时，用户会先看到图像的顶部，然后更多的图像将在图像逐步加载时显示。无损JPEG与基线JPEG类似，但具有更小的压缩比。


<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image6.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image6.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image6.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image6.jpg"
        alt="baseline JPEGs load top to bottom" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image6.jpg"/>
</noscript>
</picture>
<figcaption>基线JPEG是从顶部到底部加载，而渐进式JPEG是从模糊到清晰。</figcaption>
</figure>

而渐进式JPEG则是将图片进行多次扫描。第一次扫描会以模糊或低质量显示图片，后面多次扫描可逐步提高图像质量，这便是“渐进式”的意义。图像的每个“扫描”增加了更多的图片细节。最终组合时，就会创建一个最终质量的图像。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image7.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image7.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image7.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image7.jpg"
        alt="progressive JPEGs load from low-resolution to high-resolution" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image7.jpg"/>
</noscript>
</picture>
<figcaption>基线JPEG从上到下加载图像，而PJPEG从低分辨率（模糊）加载到高分辨率。Pat Meenan曾经写过写了一个[交互式工具](http://www.patrickmeenan.com/progressive/view.php?img=https%3A%2F%2Fwww.nps.gov%2Fplanyourvisit%2Fimages%2FGrandCanyonSunset_960w.jpg)来测试和分析渐进式JPEG扫描。</figcaption>
</figure>

无损JPEG是通过删除由数码相机或编辑器添加的[EXIF数据](http://www.verexif.com/en/)、优化一个图片的[霍夫曼编码](https://en.wikipedia.org/wiki/Huffman_coding)或者重新扫描图像等手段实现的。诸如[jpegtran](http://jpegclub.org/jpegtran/)等工具都可以通过重新排列压缩数据而无需图像降级来实现无损压缩。[jpegrescan](https://github.com/kud/jpegrescan)，[jpegoptim](https://github.com/tjko/jpegoptim)和[mozjpeg](https://github.com/mozilla/mozjpeg)（我们将在稍后介绍）同样也支持无损JPEG压缩。


### <a id="the-advantages-of-progressive-jpegs" href="#the-advantages-of-progressive-jpegs">渐进式JPEG的优点</a>

PJPEGs能提供图像低分辨率“预览”功能，可以提高用户的使用体验：用户会感觉图像的加载速度更快。

在较慢的3G网络连接中，这允许用户在只收到一部分文件时就可以（大概）看到图像中的内容，并确定是否等待它完全加载完成。这会比基线JPEG所提供的图像从上到下显示方式让人更加乐于接受。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1504993129/essential-image-optimization/pjpeg-graph.png"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1504993129/essential-image-optimization/pjpeg-graph.png"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504993129/essential-image-optimization/pjpeg-graph.png" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504993129/essential-image-optimization/pjpeg-graph.png"
        alt="impact to wait time of switching to progressive jpeg" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504993129/essential-image-optimization/pjpeg-graph.png"/>
</noscript>
</picture>
<figcaption>在2015年，[Facebook更新图像到PJPEG（用于iOS应用程序）](https://code.facebook.com/posts/857662304298232/faster-photos-in-facebook-for-ios/)后，数据使用量下降了10％。通过优化感知加载时间，Facebook能够比快15％的速度加载优质图像，如上图所示。</figcaption>
</figure>

PJPEG还可以提高图像压缩率；同样10KB以上的图像，渐进式JPEG比基线JPEG可以节省[2-10％](http://www.bookofspeed.com/chapter5.html)的带宽。渐进式JPEG有较高压缩比，是由于JPEG中的每个扫描都能够拥有自己专用的可选[霍夫曼编码](https://en.wikipedia.org/wiki/Huffman_coding)。现代化的JPEG编码器（例如[libjpeg-turbo](http://libjpeg-turbo.virtualgl.org/)，MozJPEG等）都是利用了PJPEG的灵活性，更好地打包图像数据。

<aside class="note"><b>注意:</b> 为什么PJPEG的压缩更好？因为基线JPEG的所有块是一次性压缩编码的。而在PJPEG中，利用一种类似[离散余弦变换](https://en.wikipedia.org/wiki/Discrete_cosine_transform)系数的方法可以将多个不连续的块编码在一起，从而带来更好的压缩比率。</aside>

### <a id="whos-using-progressive-jpegs-in-production" href="#whos-using-progressive-jpegs-in-production">谁在生产环境中使用了渐进式JPEG？</a>

*   [Twitter.com](https://www.webpagetest.org/performance_optimization.php?test=170717_NQ_1K9P&run=2#compress_images)：推特使用了质量基准为85%的渐进式JPEG。通过他们对用户延迟感知（首次扫描的时间和总体加载时间）的测试显示，总体而言，PJPEG在解决他们对较低的文件大小和可接受范围内的转码及解码时间的需求方面具有很强的竞争力。
*   [Facebook](https://code.facebook.com/posts/857662304298232/faster-photos-in-facebook-for-ios/)：脸书为他们的iOS应用使用了渐进式JPEG。他们发现数据的使用量减少了10％，并且他们载入高质量图像的速度提高了15％。
*   [Yelp](https://engineeringblog.yelp.com/2017/06/making-photos-smaller.html)：在切换到渐进式JPEG后，发现他们的图像尺寸减少了4.5％。另外，他们还使用MozJPEG节省了13.8％的额外费用。

包括许多以图片为核心业务的网站，比如[Pinterest](https://pinterest.com/)同样都是在生产环境中使用渐进式JPEG。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1506382472/essential-image-optimization/pinterest-loading.png"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1506382472/essential-image-optimization/pinterest-loading.png"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1506382472/essential-image-optimization/pinterest-loading.png" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1506382472/essential-image-optimization/pinterest-loading.png"
        alt="Pinterests JPEGs are all progressively encoded. This optimizes the user experience by loading them each scan-by-scan." />
<noscript>
  <img src="http://res.cloudinary.com/ddxwdqwkr/image/upload/v1506382472/essential-image-optimization/pinterest-loading.png
"/>
</noscript>
</picture>
<figcaption>Pinterest的JPEG图像都使用了渐进式模式的编码。这可以通过多次扫描来加载来，优化用户体验。</figcaption>
</figure>

### <a id="the-disadvantages-of-progressive-jpegs" href="#the-disadvantages-of-progressive-jpegs">渐进式JPEG的缺点</a>

**PJPEG解码速度更慢**：有时候甚至多出3倍的时间。在具有强大CPU的台式机上，这种差异可能无法感觉到，但在性能有限的移动设备上就会很明显。为了不完整的图层可以正常工作，这需要对图像进行多次解码，而每次解码都需要消耗一定的CPU运算周期。

**渐进式JPEG也并不总是更小**：对于一些非常小的图像（如缩略图），渐进式JPEG的文件可能反而会大于基线JPEG。而且，对于这样小的缩略图，渐进渲染并没有什么意义。

这意味着，你在决定是否使用PJPEG时，需要进行一些测试，并找到文件大小、网络延迟和CPU周期使用的平衡点。

注意: PJPEG（和所有JPEG）有时可以在移动设备上进行硬件解码。它不会改善图片在内存中的大小，但它可以解决一些CPU负载的问题。但是并不是所有的Android设备都具有硬件加速支持；但是高端设备都会支持，所有的iOS设备也都支持。

有些用户可能会认为渐进式加载是一个缺点，因为当图像完成加载时可能会变得很难分辨。由于每个受众的感官差异很大，在使用PJPEG时请评估它对自己用户的体验是否有意义。

### <a id="how-to-create-progressive-jpegs" href="#how-to-create-progressive-jpegs">如何生成一个渐进式JPEG？</a>

很多工具和库都支持导出渐进式JPEG，比如[ImageMagick](https://www.imagemagick.org/), [libjpeg](http://libjpeg.sourceforge.net/), [jpegtran](http://jpegclub.org/jpegtran/), [jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive)以及[imagemin](https://github.com/imagemin/imagemin)。如果你已经有了一个现有的图片优化方案，那么增加渐进式图片加载支持可能是非常容易的：

```js
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');

gulp.task('images', function () {
    return gulp.src('images/*.jpg')
        .pipe(imagemin({
            progressive: true
        }))
        .pipe(gulp.dest('dist'));       
});
```

大多数的图像编辑工具默认情况下都是将图像保存为基线式。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_crop,h_477,w_2128,y_0/c_scale,w_500,/v1502426282/essential-image-optimization/photoshop.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_crop,h_477,w_2128,y_0/c_scale,w_900/v1502426282/essential-image-optimization/photoshop.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_crop,h_477,w_2128,y_0/v1502426282/essential-image-optimization/photoshop.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_crop,h_477,w_2128,y_0/v1502426282/essential-image-optimization/photoshop.jpg"
        alt="photoshop supports exporting to progressive jpeg from the file export menu" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_crop,h_477,w_2128,y_0/v1502426282/essential-image-optimization/photoshop.jpg"/>
</noscript>
</picture>
<figcaption>默认情况下，大多数图像编辑工具将图像保存为基线JPEG文件。在**Photoshop**中您可以通过选择文件 - >导出 - >保存为Web（旧版），然后选中**Progressive**选项，将您在Photoshop中创建的任何图像保存为渐进式JPEG。**Sketch**也支持导出渐进式JPEG。通过选择导出为JPG，并在保存图像的同时选中“Progressive”复选框。</figcaption>
</figure>

### <a id="chroma-subsampling" href="#chroma-subsampling">色度（或颜色）抽样</a>

我们的眼睛相对于图像中的光亮度（或者luma—一种光亮度量单位）损失来说，更易忽略图像（色度）中的颜色细节的损失。[色度抽样](https://en.wikipedia.org/wiki/Chroma_subsampling)是一种压缩方式，通过在色差通道上进行较低（相对亮度通道）清晰度的抽样从而可以减少文件大小，在某些情况下可减少[15-17％](https://calendar.perfplanet.com/2015/why-arent-your-images-using-chroma-subsampling/)，而不会对图像质量产生影响，这是JPEG图像的一个压缩选项。抽样同时也可以减少显示图像时对内存的占用。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1503683718/essential-image-optimization/luma-signal.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1503683718/essential-image-optimization/luma-signal.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503683718/essential-image-optimization/luma-signal.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503683718/essential-image-optimization/luma-signal.jpg"
        alt="signal = chroma + luma" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503683718/essential-image-optimization/luma-signal.jpg"/>
</noscript>
</picture>
</figure>

同我们看到的图片中的形状相比，定义它的亮度（luma）反而是非常重要的。比较旧的或经过过滤的黑白照片可能并不包含颜色，但是由于亮度的作用，这些照片与有颜色的照片一样能表现各种细节。色度（色彩）其实对视觉感受影响是较小的。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1503683936/essential-image-optimization/no-subsampling.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1503683936/essential-image-optimization/no-subsampling.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503683936/essential-image-optimization/no-subsampling.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503683936/essential-image-optimization/no-subsampling.jpg"
        alt="JPEG includes support for numerous subsampling types: none, horizontal and horizontal and vertical." />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503683936/essential-image-optimization/no-subsampling.jpg"/>
</noscript>
</picture>
<figcaption>JPEG支持多种不同的子采样类型，包括：无、水平和水平并垂直。这个图是来自FrédéricKayser的“[JPEGs for the horseshoe crabs](http://frdx.free.fr/JPEG_for_the_horseshoe_crabs.pdf)”。</figcaption>
</figure>

在谈论色度抽样时，经常会提到一些常见的样本。一般来说有`4:4:4`，`4:2:2`和`4:2:0`。那么这些代表什么呢？假设样本采用格式A：B：C。A是一行中的像素数，对于JPEG来说这个值一般为4；B表示第一行中的颜色数量；C表示第二行中的颜色数量。

- 4:4:4 —— 没有压缩，颜色和亮度被完全传输。
- 4:2:2 —— 水平半抽样，垂直全部抽样。
- 4:2:0 —— 第一行像素中的一半取样，并第二行忽略。

> **注意**：jpegtran和cjpeg支持单独的亮度和色度的质量配置。通过添加`-sample`标志（例如`-sample 2x1`）。并且有一些常用的规则：subsampling（`-sample 2x2`）可以生成美妙的照片；no-subsampling（`-sample 1x1`）最适合于截图、横幅和按钮；最终的compromise（`2x1`）用于你不确定将图片使用在哪里。

通过减少色度分量中的像素，可以显著减小颜色分量的大小，从而最终减少图像文件的字节大小。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1503684781/essential-image-optimization/subsampling.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1503684781/essential-image-optimization/subsampling.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503684781/essential-image-optimization/subsampling.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503684781/essential-image-optimization/subsampling.jpg"
        alt="Chrome subsampling configurations for a JPEG at quality 80." />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503684781/essential-image-optimization/subsampling.jpg"/>
</noscript>
</picture>
<figcaption>质量为80的JPEG在Chrome中不同样本配置下的显示对比</figcaption>
</figure>

色度抽样值得被用于大多数类型的图像。但是它确实有一些例外：由于抽样依赖于我们眼中的视觉限制，当对于压缩图像而言，色彩细节可能与亮度（例如医学图像）一样重要时，使用色度抽样就不是很好。

另外，包含字体的图像也可能会受到明显的影响，因为文本的抽样可能会降低其可读性。而一些更为锐利的边缘也是难以被压缩的，因为色度取样是被设计为更好地处理具有更多软过渡的摄影场景。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1503684410/essential-image-optimization/Screen_Shot_2017-08-25_at_11.06.27_AM.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1503684410/essential-image-optimization/Screen_Shot_2017-08-25_at_11.06.27_AM.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503684410/essential-image-optimization/Screen_Shot_2017-08-25_at_11.06.27_AM.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503684410/essential-image-optimization/Screen_Shot_2017-08-25_at_11.06.27_AM.jpg"
        alt="Be careful when using heavy subsampling with images containing text" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503684410/essential-image-optimization/Screen_Shot_2017-08-25_at_11.06.27_AM.jpg"/>
</noscript>
</picture>
<figcaption>“[了解JPEG](http://compress-or-die.com/Understanding-JPG/)”中建议在处理包含文本的图像时，使用4:4:4（1x1）的子样本配置。</figcaption>
</figure>

提示：在JPEG规范中其实并没有规定Chroma子采样的确切方法，因此不同的解码器的处理方式可能不同。MozJPEG和libjpeg-turbo都是使用了缩放方法。而较旧版本的libjpeg使用了不同方法：在颜色中增加边缘震荡效应。

<aside class="note"><b>注意:</b> Photoshop在使用“Save for web”功能时是自动设置色度抽样样本的。当图像质量设置在51-100之间时，根本不使用抽样（即样本`4:4:4`）。当质量低于此值时，将使用`4:2:0`进行抽样。这也就是将质量从51切换到50时，可以明显观察到的文件大小减少的一个原因。</aside>

<aside class="note"><b>注意:</b> 在抽样讨论中，常常提到[YCbCr](https://en.wikipedia.org/wiki/YCbCr)这个术语。这是一个可以表示伽玛校正的[RGB](https://en.wikipedia.org/wiki/RGB_color_model)颜色空间的模型。Y是伽马校正亮度，Cb是蓝色的色度分量，Cr是红色的色度分量。如果您查看ExifData，您将在采样级别旁边看到YCbCr。</aside>

如果想进一步了解色度抽样，请参阅《[为什么你的图像不使用色度抽样？](https://calendar.perfplanet.com/2015/why-arent-your-images-using-chroma-subsampling/)》

### <a id="how-far-have-we-come-from-the-jpeg" href="#how-far-have-we-come-from-the-jpeg">JPEG引发的格式拓展</a>

**以下是网络上当前图像格式的情况：**

*总的来说——碎片化严重。我们经常需要有条件地为不同的浏览器提供不同的图像格式，以充分利用浏览器任何先进的功能。*

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/format-comparison.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/format-comparison.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_95/v1502426282/essential-image-optimization/format-comparison.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_95/v1502426282/essential-image-optimization/format-comparison.jpg"
        alt="modern image formats compared based on quality." />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_95/v1502426282/essential-image-optimization/format-comparison.jpg"/>
</noscript>
</picture>
<figcaption>不同的现代图像格式（和优化编码器）文件大小为26KB的对比演示。我们可以使用[SSIM](https://en.wikipedia.org/wiki/Structural_similarity)（结构相似性）或[Butteraugli](https://github.com/google/butteraugli)来比较质量，稍后我们将会详细介绍。</figcaption>
</figure>

*   **[JPEG 2000](https://en.wikipedia.org/wiki/JPEG_2000) (2000)** - 从基于离散余弦变换到基于小波的切换方式改进。**浏览器支持：Safari桌面+ iOS**。**
*   **[JPEG XR](https://en.wikipedia.org/wiki/JPEG_XR) (2009)** - 替代JPEG和JPEG 2000，支持[ HDR](http://wikivisually.com/wiki/High_dynamic_range_imaging)和宽[色域](http://wikivisually.com/wiki/Gamut)色彩空间。编码/解码速度比较慢，但是产生比JPEG更小的文件。**浏览器支持：Edge，IE。**
*   **[WebP](https://en.wikipedia.org/wiki/WebP) (2010)** - Google支持的基于块预测的图像格式，包括有损和无损压缩。WebP同时提供JPEG的字节压缩特性和一般文件较大PNG才能提供的透明度特性。但是不支持色度抽样和渐进式加载。另外解码也比JPEG解码速度慢。**浏览器支持：Chrome，Opera。通过Safari和Firefox进行实验。**
*   **[FLIF](https://en.wikipedia.org/wiki/Free_Lossless_Image_Format) (2015)** - 基于压缩比的一种无损图像格式，并声称胜过PNG、无损WebP、无损BPG和无损JPEG 2000。**浏览器支持：无。注意，有一个[JS浏览器解码器](https://github.com/UprootLabs/poly-flif)。**
*   **HEIF和BPG。**从压缩的角度来看，它们是一样的，但是有不同的包装:
*   **[BPG](https://en.wikipedia.org/wiki/Better_Portable_Graphics) (2015)** - 旨在基于HEVC（[高效率视频编码](http://wikivisually.com/wiki/High_Efficiency_Video_Coding)）进行高效压缩的JPEG替代者。与MozJPEG和WebP相比，可以压缩出更小的文件。由于许可证问题，不太可能获得广泛的支持。**浏览器支持：无。注意，有一个*[JS浏览器解码器](https://bellard.org/bpg/)。***
*   **[HEIF](https://en.wikipedia.org/wiki/High_Efficiency_Image_File_Format) (2015)** - 用于存储HEVC编码的图像和图像序列的一种格式。苹果公司在[ WWDC](https://www.cnet.com/news/apple-ios-boosts-heif-photos-over-jpeg-wwdc/)宣布，他们将在iOS上将JPEG转换为HEIF，从而节省了2倍的文件存储空间。**浏览器支持：无，在文章创建时。现在Safari（Mac和iOS 11）可以支持。**

如果你想更直观的了解，你可以欣赏一些上述格式的视觉比较工具，比如[这里](https://people.xiph.org/~xiphmont/demo/daala/update1-tool2b.shtml)或[这里](http://xooyoozoo.github.io/yolo-octo-bugfixes/#cologne-cathedral&jpg=s&webp=s)。

综上所述可以看到，**浏览器对图像格式的支持是很分散的**。如果你希望利用上述任何一种格式，那么你就可能需要有条件地为每个目标浏览器提供不同的返回内容。在Google，我们已经看到了对WebP的一些支持，所以我们后面会做更深入的介绍。

您还可以使用.jpg扩展名（或任何其他扩展名）来表示一个图像格式（例如WebP，JPEG 2000），因为浏览器可以决定它渲染图像的媒体类型。这就允许服务器端使用请求中的[Content-Type设置](https://www.igvita.com/2012/12/18/deploying-new-image-formats-on-the-web/)来决定要发送哪种格式的图像，而无需更改HTML中的内容。像Instart Logic这样的服务商在向他们的客户传送图像时，都是使用的这种方法。

接下来，让我们讨论另一种情况，当你无法有条件地提供不同的图像格式时使用的方法：使用**JPEG优化编码器**工具。


### <a id="optimizing-jpeg-encoders" href="#optimizing-jpeg-encoders">使用JPEG优化编码器</a>

现代化的JPEG编码器会尝试生产出更小更高保真的JPEG文件，同时保持与现有浏览器以及图像处理应用程序的兼容性。它们避免了在运行的系统中引入新的图像格式而带来一系列变化，并给图像优化压缩带来质的变化。这里就有两个这样的编码器，它们是MozJPEG和Guetzli。

***简单聊一下：你该使用哪个编码器？***

* 一般使用MozJPEG
* 如果你的核心关注点是图像质量，而且不在乎相对较长的编码时间，则使用Guetzli
* 如果你需要更好的可配置性:
 * [JPEGRecompress](https://github.com/danielgtaylor/jpeg-archive) （底层使用了MozJPEG）
 * [JPEGMini](http://www.jpegmini.com/)：它类似于Guetzli会自动选择最佳质量。但它不如Guetzli技术复杂，而且更快，其目标是在于更适用于网络的最佳质量图像。
 * [ImageOptim API](https://imageoptim.com/api)（[这里](https://imageoptim.com/online)还有个免费的在线界面）：它的颜色处理机制是独一无二的，您可以分开选择颜色质量与整体质量。另外，它可以自动选择色度抽样级别，以便在屏幕截图中保留高分辨率颜色，并避免了在自然照片中平滑色彩的字节浪费。


### <a id="what-is-mozjpeg" href="#what-is-mozjpeg">什么是MozJPEG？</a>

[MozJPEG](https://github.com/mozilla/mozjpeg)是Mozilla开源提供的现代化的JPEG编码器。它[声称](https://research.mozilla.org/2014/03/05/introducing-the-mozjpeg-project/)要去除JPEG文件的10％左右的体积。使用MozJPEG压缩的JPEG文件可以跨浏览器显示，并且还包括渐进式扫描优化、[网格量化](https://en.wikipedia.org/wiki/Trellis_quantization)（丢弃最少压缩的细节）以及一些优秀的[量化预设](https://calendar.perfplanet.com/2014/mozjpeg-3-0/)等功能，可以帮助创建更平滑的高分辨率图像（当然也可以使用ImageMagick，如果你愿意去浏览XML配置）。

[MozJPEG](https://github.com/ImageOptim/ImageOptim/issues/45)不仅被[ImageOptim](https://github.com/ImageOptim/ImageOptim/issues/45)所支持，并且还有一个可靠的可配置[imagemin](https://github.com/imagemin/imagemin-mozjpeg)插件。以下是Gulp的简单实现示例：

```js
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');
const imageminMozjpeg = require('imagemin-mozjpeg');

gulp.task('mozjpeg', () =>
    gulp.src('src/*.jpg')
    .pipe(imagemin([imageminMozjpeg({
        quality: 85
    })]))
    .pipe(gulp.dest('dist'))
);
```

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image10.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image10.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image10.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image10.jpg"
        alt="mozjpeg being run from the command-line" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image10.jpg"/>
</noscript>
</picture>
</figure>


<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image11.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image11.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image11.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image11.jpg"
        alt="mozjpeg compression at different qualities. At q=90, 841KB. At q=85, 562KB. At q=75, 324KB. Similarly, Butteraugli and SSIM scores get slightly worse as we lower quality." />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image11.jpg"/>
</noscript>
</picture>
<figcaption>MozJPEG：不同质量的文件体积与视觉相似度得分的比较。</figcaption>
</figure>

我用来自[jpeg-archive](https://github.com/danielgtaylor/jpeg-archive)项目的[jpeg-recompress](https://github.com/imagemin/imagemin-jpeg-recompress)来计算一个源图像的SSIM（结构相似性）分数。SSIM是一种用于测量两个图像之间的相似性的方法，其中的分数就是一个图像对比另一个图像的“完美”相似测量值。

根据我的测量结果，MozJPEG的确是一个很好的选择，它可以保持高视觉质量来压缩网页的图像，同时减少文件的大小。对于中小型图像，我发现MozJPEG（质量= 80-85）可以将文件大小减少30-40％，同时保持可接受的SSIM，在jpeg-turbo库上可以提高了5-6％读取速度。MozJPEG确实要比基线JPEG[编码更慢](http://www.libjpeg-turbo.org/About/Mozjpeg)，但这不足以成为你放弃它的理由。

<aside class="note"><b>注意:</b> 如果您需要一个支持MozjPEG的工具，并包含配置支持，以及一些免费的图像比较实用程序，请查看[jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive)。《Web Performance in Action》的作者Jeremy Wagner提供了一种参考[配置](https://twitter.com/malchata/status/884836650563579904)，如下：

```
jpeg-recompress --min 35 --max 70 --strip --method smallfry --loops 16 in.jpg out.jpg
```

</aside>

### <a id="what-is-guetzli" href="#what-is-guetzli">什么是Guetzli？</a>

[[Guetzli](https://github.com/google/guetzli)是一个来自谷歌的、有前景的、有些缓慢的感知型的JPEG编码器，它会试图找到一个人眼在视觉上无法区分差异但却体积最小的JPEG文件。Guetzli会执行一系列感知测试，为最终的JPEG提出方案，并对每个方案进行评估。最终在其中选择最高评分的提案作为最终输出。

而为了测量图像之间的差异，Guetzli使用[Butteraugli](https://github.com/google/butteraugli)，一种基于人类感知来测量图像差异的模型（下面会介绍）。Guetzli可以考虑到其他JPEG编码器没有的几个视觉属性。例如，人眼所看到的绿光量与蓝色的敏感度之间是存在关系的，因此绿色旁边的蓝色信息的编码就会动态修改的更精准一些。

<aside class="note"><b>注意:</b> 图片文件的大小更多取决于你图片的质量，而非使用的编解码器。与通过切换编解码器实现的文件大小节省相比，最低质量和最高质量JPEG之间的文件大小差异要更大的多。因此，设置最低可接受的质量非常重要。避免将您的质量设置得太高，并且不去关注它。</aside>

Guetzli [声称](https://research.googleblog.com/2017/03/announcing-guetzli-new-open-source-jpeg.html)当在Butteraugli得到同样的质量评分的情况下，和其他编码器相比可以将数据体积再减小20-30％。使用Guetzli的一个大问题就是，它非常非常慢，目前只适用于静态内容的优化。从它的说明中可以看出，Guetzli需要大量的内存（每百万像素可能需要1分钟+ 200MB的内存）。这里有一个很好的Guetzli[实践测试报告](https://github.com/google/guetzli/issues/50)。可以看到，Guetzli可以作为你的静态网站构建过程中图片优化的理想拼图，但是在按需执行时就并不合适了。

<aside class="note"><b>注意:</b> 当您将图像优化作为静态网站的构建过程的一部分时，Guetzli可能更加适用，或者在不需要按照要求执行图像优化的情况时。</aside>

像ImageOptim这样的工具，同样支持Guetzli优化（在[最新版本中](https://imageoptim.com/)）。

```js
const gulp = require('gulp');
const imagemin = require('gulp-imagemin');
const imageminGuetzli = require('imagemin-guetzli');

gulp.task('guetzli', () =>
    gulp.src('src/*.jpg')
    .pipe(imagemin([
        imageminGuetzli({
            quality: 85
        })
    ]))
    .pipe(gulp.dest('dist'))

);
```

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image12.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image12.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image12.jpg" />

<img
        class="small lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image12.jpg"
        alt="guetzli being run from gulp for optimisation" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image12.jpg"/>
</noscript>
</picture>
</figure>

可以看到，为了节省些空间，使用Guetzli编码一张3 x 3MP的图像要花费近七分钟的时间（并且伴随着高CPU使用率）。但是为了存档更高分辨率的照片，我认为付出些代价也是值得的。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image13.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image13.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image13.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image13.jpg"
        alt="comparison of guetzli at different qualities. q=100, 945KB. q=90, 687KB. q=85, 542KB." />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image13.jpg"/>
</noscript>
</picture>
<figcaption>Guetzli：不同质量的文件体积和视觉相似度得分的比较。</figcaption>
</figure>


<aside class="note"><b>注意:</b>建议在高质量图像（例如未压缩的图像，PNG原图或者100％质量（或无损）的JPEG）上使用Guetzli。虽然它可以在其他质量的图像（例如质量为84或更低的JPEG）上工作，但结果可能较差。</aside>

使用Guetzli压缩图像是非常（非常）耗时的，可能会使你的用户转身而去，但是对于较大的图像，这是值得的。我已经可以看到一些使用案例里，Guetzli可以仅仅保留文件大小的40％，同时保持了视觉的保真度。这使它非常适合存档照片。在小到中等大小的图像上，我仍然看到一些地方在应用（在10-15KB的范围内），但是效果并不显著。Guetzli可以在压缩更小的图像上引入更多的液化曲线失真。

您可能还对Eric Portis的这项[研究](https://cloudinary.com/blog/a_closer_look_at_guetzli)感兴趣，他将Guetzli与Cloudinary的自动压缩功能进行了比较，并从中获得了一些有效的不同数据点。

### <a id="mozjpeg-vs-guetzli" href="#mozjpeg-vs-guetzli">MozJPEG与Guetzli孰优孰劣？</a>

比较不同的JPEG编码器是很复杂的 —— 它需要比较压缩图像的质量、保真度以及最终文件大小等多项内容。正如图像压缩专家KornelLesiński[指出](https://kornel.ski/faircomparison)的那样，仅就单方面而非多个角度进行测试比较，很可能会导致一个无效的结论。

那Guetzli和MozJPEG比究竟如何呢？—— Kornel指出：

- Guetzli更适用于更高品质的图像（butteraugli建议最好是`q=90`以上，而MozJPEG更适宜处理`q=75`左右的图像）
- Guetzli压缩速度要慢得多（都是生成了标准的JPEG，所以解码速度是一样很快的）
- MozJPEG不会自动选择质量设置，但您可以使用外部工具找到最佳质量，例如[jpeg-archive](https://github.com/danielgtaylor/jpeg-archive)

存在多种方法，都可以用于测定压缩图像与原图像的相似度或视觉感知差异度。一些图像质量研究经常会使用像[SSIM](https://en.wikipedia.org/wiki/Structural_similarity)（结构相似性）这样的方法。然而，Guetzli则是通过优化Butteraugli的方法来实现的。

### <a id="butteraugli" href="#butteraugli">Butteraugli</a>

[[Butteraugli](https://github.com/google/butteraugli)是一个来自Google的项目，它可以估算一个人可能会注意到两个图像的视觉降级（即心理视觉相似性）的点，并给出几乎没有区别的两个图像的比对分数。Butteraugli不仅给出一个标量的分数，而且还会计算出图像差异水平的空间图。所以当SSIM专注于计算图像中差异的总和时，Butteraugli则更专注于差异最明显的部分。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image14.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image14.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_95/v1502426282/essential-image-optimization/Modern-Image14.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_95/v1502426282/essential-image-optimization/Modern-Image14.jpg"
        alt="butteraugli validating an image of a parrot" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_95/v1502426282/essential-image-optimization/Modern-Image14.jpg"/>
</noscript>
</picture>
<figcaption>上图是一个例子：使用Butteraugli找到用户无法注意到视觉差异的最小质量阈值。并且缩小了65%的文件体积。</figcaption>
</figure>


在真正的实施中，您将会制定一个图像质量的目标，然后运行一些不同的图像优化策略，查看您的Butteraugli分数，然后再找到合适的文件大小与质量级别的最佳平衡点。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image15.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image15.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image15.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image15.jpg"
        alt="butteraugli being run from the command line" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image15.jpg"/>
</noscript>
</picture>
<figcaption>总而言之，我花费了30分钟来安装Butteraugli到我的Mac上，包括安装Bazel和编译C++的源码。使用它就非常简单了：选择两个图片（一个原图和一个压缩版本）进行比较，它就会给你一个比较分数。 </figcaption>
</figure>

**Butteraugli与其他视觉相似度比较算法有什么不同？**

[根据一位Guetzli项目成员的这条[评论](https://github.com/google/guetzli/issues/10#issuecomment-276295265)表明，Guetzli在Butteraugli得分最高，但在SSIM和MozJPEG得分也最差。其实，这是符合我自己对图像优化策略的研究的。我会运行Butteraugli和一个Node模块（如[img-ssim](https://www.npmjs.com/package/img-ssim)）比较在使用了Guetzli、MozJPEG的之前和之后，源图像和压缩图像的SSIM分数。

**组合编码器？**

对于一些较大的图像，我发现将Guetzli与MozJPEG中的**无损压缩**（jpegtran，而不是cjpeg，避免丢掉了Guetzli完成的工作）结合起来使用，可以将文件大小再减少10~15%（总体55%），并且只有非常小的SSIM评分损失。我只是提醒一下可以在组合使用编码器方面进行一些试验和分析，但是也受到了[Ariya Hidayat](https://ariya.io/2017/03/squeezing-jpeg-images-with-guetzli)等业内其他人的[好评](https://ariya.io/2017/03/squeezing-jpeg-images-with-guetzli)。

总结来说，MozJPEG是一个初学者友好的网页资源编码器，速度相对较快，可以生成高质量的图像。而Guetzli则是一个资源密集型的编码器，它在较大、更高质量的图像上效果最好，是我建议给中高级用户的一个好选择。

## <a id="what-is-webp" href="#what-is-webp">什么是WebP？</a>

[WebP](https://developers.google.com/speed/webp/)是一个来自Google的新型图像格式，它旨在以可接受的视觉质量提供较低文件大小的无损和有损压缩。另外，WebP还包括了对alpha通道透明度和动画的支持。

在过去一年中，WebP在有损和无损模式下减少了10%的压缩体积，并且提高了10%的压缩速度。WebP并不是一个能够用于所有目的的工具，但它在图像压缩社区中已经具有了一定的地位和不断增长的用户群。那么，我们来看看为什么。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image16.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image16.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image16.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image16.jpg"
        alt="comparison of webp at different quality settings. q=90, 646KB. q=80= 290KB. q=75, 219KB. q=70, 199KB" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/Modern-Image16.jpg"/>
</noscript>
</picture>
<figcaption>WebP：不同质量级别下文件体积和相似度对比。</figcaption>
</figure>

### <a id="how-does-webp-perform" href="#how-does-webp-perform">WebP的表现如何？</a>

**有损压缩**

根据WebP团队的[描述](https://developers.google.com/speed/webp/docs/webp_study)，使用VP8或VP9视频关键帧编码进行处理的WebP压缩文件平均比JPEG文件小了25-34％。

在较低质量设置（0~50）下，WebP具有比JPEG更大的优势，因为它可以消除丑陋的块状伪影；在中等质量设置（-m 4 -q 75）下，WebP则是速度与体积平衡的默认选择；而在较高质量设置（80-99）时，WebP的优势则在缩小。WebP被推荐应用在速度比质量更重要的场景中。

**无损压缩**

[WebP无损文件的体积比PNG文件小了26％](https://developers.google.com/speed/webp/docs/webp_lossless_alpha_study)；同时，WebP无损压缩图片的加载时间与PNG相比减少了3％。也就是说，您通常不会想在网络上为您的用户提供无损压缩的图像。另外，无损和锐利边缘（如non-JPEG）是不同的。无损模式的WebP可能更适合于档案内容。

**透明度**

WebP具有一个无损8位透明度通道，而只比PNG多出22％的字节。它还支持有损的RGB透明度，这是WebP独有的功能。

**元数据**

WebP文件格式支持EXIF照片元数据和XMP数字文档元数据，它还包含了一个ICC颜色配置文件。

WebP提供了更好的压缩，但代价是更多的CPU开销。在2013年，WebP的压缩速度会比JPEG慢10倍，但是现在已经可以忽略不计了（有些图像可能会减慢2倍）。当WebP用于生产作为你应用构建一部分的静态图像时，这不应该是一个大问题。然而，作为动态的图像生产工具，就可能会导致可察觉的CPU开销，这将是您需要考量的问题。

<aside class="note"><b>注意:</b> WebP有损质量设置百分比，与JPEG的质量百分比并不能直接比较。因为WebP通过丢弃更多数据来实现更小的文件体积，因此“70％质量”的JPEG将与“70％质量”的WebP图像完全不同。</aside>

### <a id="whos-using-webp-in-production" href="#whos-using-webp-in-production">谁在生产环境中使用WebP？</a>

许多大型公司正在生产环境中使用WebP，来降低成本并减少网页的加载时间。

Google的报告表明，以一天提供430亿图像请求来计算，使用WebP有损压缩将比其他有损压缩解决方案节省30%~35%的存储空间，而使用无损压缩则是节省26%。这种空间的节省无疑是非常重要的。而如果[浏览器对WebP支持](http://caniuse.com/#search=webp)更好、更广泛，节省将会更大。Google已经将WebP应用于Google Play和YouTube等网站的生产环境。

Netflix、亚马逊、Quora、雅虎、沃尔玛、Ebay、“卫报”、“财富”和“今日美国”均是为已经支持格式的浏览器提供了WebP。VoxMedia 通过给他们使用Chrome的用户提供WebP，将The Verge网站（一个美国科技媒体网站） [减少了1~3倍的加载时间](https://product.voxmedia.com/2015/8/13/9143805/performance-update-2-electric-boogaloo)。[500px](https://iso.500px.com/500px-color-profiles-file-formats-and-you/)则表示，通过切换为WebP提供给自己的Chrome用户，在保持类似或更好的图像质量的同时将文件的体积平均降低的25%。

这个样本列表显示，有很多的公司都使用WebP。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/webp-conversion.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/webp-conversion.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/webp-conversion.jpg" />

<img
        class="small lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/webp-conversion.jpg"
        alt="WebP stats at Google: over 43B image requests a day" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/webp-conversion.jpg"/>
</noscript>
</picture>
<figcaption>WebP在Google的应用：在YouTube、Google Play、Chrome的Data Saver以及G+（Google+，Google的社交网络服务）上，每日响应430亿的WebP请求。</figcaption>
</figure>

### <a id="how-does-webp-encoding-work" href="#how-does-webp-encoding-work">WebP编码如何执行？</a>

WebP的有损编码被设计用于与JPEG静态图像进行竞争。它有三个关键流程：

**宏块分解**：将图像分解成一个16×16亮度像素（宏）块集合和两个8×8色度像素块集合。这可能听起来很像JPEG的方法，通过色彩空间转换、色度通道下采样和图像细分来处理。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image18.png"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image18.png"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image18.png" />

<img
        class="small lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image18.png"
        alt="Macro-blocking example of a Google Doodle where we break a range of pixels down into luma and chroma blocks."/>
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image18.png"/>
</noscript>
</picture>

</figure>

**预测**：宏块的每个4×4子块都具有一个预测模型，可以有效地进行滤波。它定义了一个块周围的两组像素，分别为A（上面的行）和L（左侧的列）。编码器会使用这两个像素块A和L，尝试填充4x4像素的测试块，并确定哪个像素块创建的测试块最接近原始块的值。Colt McAnlis在他的文章“[WebP有损耗模式的工作原理](https://medium.com/@duhroach/how-webp-works-lossly-mode-33bd2b1d0670)”中更深入地讨论了这一点。


<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image19.png"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image19.png"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image19.png" />

<img
        class="lazyload very-small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image19.png"
        alt="Google Doodle example of a segment displaying the row, target block and column L when considering a prediction model."/>
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image19.png"/>
</noscript>
</picture>

</figure>

最后会使用离散余弦变换（DCT）执行几步类似于JPEG编码的过程。之间的一个关键区别是，这里会使用一个[算术压缩器](https://www.youtube.com/watch?v=FdMoL3PzmSA&index=7&list=PLOU2XLYxmsIJGErt5rrCqaSGTMyyqNt2H)，不同于JPEG的霍夫曼编码。

如果您想深入了解WebP的压缩技术，Google开发人员的文章“ [WebP压缩技术”](https://developers.google.com/speed/webp/docs/compression)将帮助你。

### <a id="webp-browser-support" href="#webp-browser-support">WebP的浏览器支持</a>

不是所有的浏览器都支持WebP，然而根据[CanIUse.com的统计](http://caniuse.com/webp)，WebP的浏览器用户支持在全球约为74％。Chrome和Opera本地支持它。Safari，Edge和Firefox已经尝试了它，但没有在官方发行版中发布。这就将为用户获取WebP图片的任务移交给了前端开发人员，这个稍后再说。

以下是主要的浏览器和WebP的支持情况：

- Chrome：Chrome在第23版开始全面支持。
- Chrome for Android：自Chrome 50起。
- Android：自Android 4.2起。
- Opera: 从12.1版本起。
- Opera Mini：所有版本。
- Firefox：一些beta支持。
- Edge：一些beta支持。
- Internet Explorer：不支持。
- Safari：一些beta支持。

WebP不是没有缺点的。它缺乏全分辨率的颜色空间选项，并且不支持逐行解码（渐进式）。总的来说，WebP工具是一个适合的、浏览器支持的（写作时仅限于Chrome和Opera浏览器）、可能会覆盖足够的用户的值得你去考虑的压缩选项。

### <a id="how-do-i-convert-to-webp" href="#how-do-i-convert-to-webp">怎样将图片转换为WebP格式?</a>

一些商业和开源的图像编辑处理软件都支持WebP格式。其中有个特别有用的应用叫做XnConvert，一个免费的、跨平台的批量图片转换器。

<aside class="note"><b>注意:</b> 一定要避免将低质量或一般质量的JPEG转换为WebP。这是一个很常见的错误，它会导致生成的WebP图片带有JPEG压缩产生的虚影效果。还会使保存的WebP图片效果变差和失真，甚至损失的质量会翻倍。作为转换初始的图像文件，一定要是高质量的，或者原图。</aside>

**[XnConvert](http://www.xnview.com/en/xnconvert/)**

XnConvert能够批量的进行图像处理，并兼容500多种图像格式。你可以组合超过80种的独立操作，以多种方式转换或编辑的图像。


<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image20.png"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image20.png"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image20.png" />

<img
        class="small lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image20.png"
        alt="XNConvert app on Mac where a number of images have been converted to WebP"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image20.png"/>
</noscript>
</picture>
<figcaption>XnConvert支持批量的图像优化，允许原图到WebP或其他格式的直接转换。除了压缩，XnConvert还可以执行图像元数据剥离、裁剪、颜色深度定制以及其他转换等操作。</figcaption>
</figure>

下面是*XnView*网站上列出的XnConvert的一些选项，包括：

*   元数据：编辑
*   转换：旋转、裁剪、调整尺寸
*   调整：亮度、对比度、饱和度
*   过滤器：模糊、浮雕、锐化
*   效果：屏蔽、水印、渐晕

使用XnConvert，你可以导出为大约70种不同的文件格式，包括WebP。它支持Linux、Mac以及Windows操作系统。因此，我们强烈推荐XnConvert，它特别适合小企业。

**Node模块**

[Imagemin](https://github.com/imagemin/imagemin)是一个非常流行的Node.js的图像缩小模块，它还包含了一个WebP转换插件（[imagemin-webp](https://github.com/imagemin/imagemin-webp)）。支持有损和无损两种模式。

要安装imagemin和imagemin-webp，需要运行：

```
> npm install --save imagemin imagemin-webp
```

然后，我们可以使用require()调用这两个模块，并且运行它们处理在项目目录中的任意图片（如JPEG）。下面是使用Imagemin，有损压缩并批量生成质量为60的WebP图片：


```js
const imagemin = require('imagemin');
const imageminWebp = require('imagemin-webp');

imagemin(['images/*.{jpg}'], 'images', {
    use: [
        imageminWebp({quality: 60})
    ]
}).then(() => {
    console.log('Images optimized');
});
```


与JPEG类似，要注意到输出后的压缩效果。评估什么样质量设置，对你的图像才是有意义的。Imagemin-webp还可通过传递`lossless: true`选项，来生成无损压缩的WebP图像（支持24位颜色和全透明度）：


```js
const imagemin = require('imagemin');
const imageminWebp = require('imagemin-webp');

imagemin(['images/*.{jpg,png}'], 'build/images', {
    use: [
        imageminWebp({lossless: true})
    ]
}).then(() => {
    console.log('Images optimized');
});
```


[这里](https://github.com/sindresorhus/gulp-webp)还有一个由Sindre Sorhus基于imagemin-webp构建的Gulp的WebP插件；当然，也包括WebPack的[插件](https://www.npmjs.com/package/webp-loader)。Gulp插件可以接收imagemin的任何选项设置，如有损压缩：

```js
const gulp = require('gulp');
const webp = require('gulp-webp');

gulp.task('webp', () =>
    gulp.src('src/*.jpg')
    .pipe(webp({
        quality: 80,
        preset: 'photo',
        method: 6
    }))
    .pipe(gulp.dest('dist'))
);
```

或无损压缩:

```js
const gulp = require('gulp');
const webp = require('gulp-webp');

gulp.task('webp-lossless', () =>
    gulp.src('src/*.jpg')
    .pipe(webp({
        lossless: true
    }))
    .pipe(gulp.dest('dist'))
);
```

**使用Bash批量优化图像**

XNConvert支持批量的图像压缩，但是如果你希望避免使用一个应用程序或者一套构建系统，那么可以使用bash命令和图像优化二进制文件，它们同样使压缩变得简单。

你可以使用[cwebp](https://developers.google.com/speed/webp/docs/cwebp)命令，将图像批量转换为WebP：

```
find ./ -type f -name '*.jpg' -exec cwebp -q 70 {} -o {}.webp \;
```

或者，使用[jpeg-recompress](https://github.com/danielgtaylor/jpeg-archive)采用MozJPEG编码，批量优化您的原图：

```
find ./ -type f -name '*.jpg' -exec jpeg-recompress {} {} \;
```

并且，使用[svgo](https://github.com/svg/svgo)优化SVG（我们稍后将介绍）：

```
find ./ -type f -name '*.svg' -exec svgo {} \;
```

Jeremy Wagner有一篇文章《[使用Bash优化图像](https://jeremywagner.me/blog/bulk-image-optimization-in-bash)》比较全面的讨论这个问题，同时还有个姊妹篇《[使用Bash快速优化图像](https://jeremywagner.me/blog/faster-bulk-image-optimization-in-bash)》也值得读一下。

**其他的WebP图像处理和编辑应用程序包括：**

   * Leptonica — 一个开源图像处理和图像分析的应用网站。
* Sketch支持直接导出WebP格式
    * GIMP — 免费、开源的Photoshop替代品。一个图像编辑器。
    * ImageMagick — 用于创建、设计、转换或编辑位图图像，一个免费的命令行应用。
    * Pixelmator — 适用于Mac的商用图像编辑器。
    * Photoshop的WebP插件  — 免费的，支持图像导入和导出，来自于Google。

**Android:** 您可以使用Android Studio将现有的BMP、JPG、PNG或静态GIF图像转换为WebP格式。有关更多信息，请参阅[使用Android Studio创建WebP图像](https://developer.android.com/studio/write/convert-webp.html)。

### <a id="how-do-i-view-webp-on-my-os" href="#how-do-i-view-webp-on-my-os">[如何在我的操作系统上浏览WebP图像？](https://images.guide/#how-do-i-view-webp-on-my-os)</a>

当你将WebP图像拖放到基于Blink的浏览器（Chrome、Opera、Brave）中进行预览时，你还可以使用Mac或Windows的附加组件直接从操作系统预览它们。

几年前，当[Facebook试验WebP](https://www.cnet.com/news/facebook-tries-googles-webp-image-format-users-squawk/)格式时发现，用户将WebP图片通过右键另存到本地磁盘后，无法通过浏览器以外的程序打开浏览。这其中主要有三个主要问题：

<ul>
<li>“另存为”但无法在本地浏览WebP图片。这个可以通过将Chrome注册为“.webp”的打开应用来解决。</li>
<li> “另存为”并将图片加入邮件中分享给不适用Chrome的其他人。Facebook通过引入一个醒目的“下载”按钮来解决这个问题，这个按钮点击时会返回JPEG给用户。</li>
<li>右键 > 复制链接 -> 分享链接到网络上。这个最后通过HTTP通讯协议中的[content-type](https://www.igvita.com/2012/12/18/deploying-new-image-formats-on-the-web/)来解决。</li>
</ul>

这些问题可能对你的用户来说不太重要，但是对于WebP图片的通用性，Facebook的试验过程可谓是一个有趣的注解。值得庆幸的是，现在不同的操作系统上，都有用于查看和使用WebP的实用程序了。

在Mac上，可以使用Quick Look的[WebP插件(qlImageSize)](https://github.com/Nyx0uf/qlImageSize)。它工作的很好：

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image22.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image22.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image22.jpg" />

<img
        class="small lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image22.jpg"
        alt="Desktop on a mac showing a WebP file previewed using the Quick Look plugin for WebP files"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image22.jpg"/>
</noscript>
</picture>
</figure>


在Windows上，您还可以下载[WebP编解码器包](https://storage.googleapis.com/downloads.webmproject.org/releases/webp/WebpCodecSetup.exe)，以便在File Explorer和Windows Photo Viewer中预览WebP图像。

### <a id="how-do-i-serve-webp" href="#how-do-i-serve-webp">[如何提供WebP？](https://images.guide/#how-do-i-serve-webp)</a>

在不支持WebP的浏览器中，WebP图像最终不会显示，这明显不是我们想要的。为了避免这种情况，我们可以使用几种策略根据浏览器支持情况有条件地提供WebP图像服务。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/play-format-webp.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/play-format-webp.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/play-format-webp.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/play-format-webp.jpg"
        alt="The Chrome DevTools Network panel displaying the waterfall for the Play Store in Chrome, where WebP is served."
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/play-format-webp.jpg"/>
</noscript>
</picture>
<figcaption>Chrome的开发者工具中的网络面板中“类型”一栏显示，网站对Blink内核的浏览器返回了WebP格式。</figcaption>
</figure>

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/play-format-type.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/play-format-type.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/play-format-type.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/play-format-type.jpg"
        alt="While the Play store delivers WebP to Blink, it falls back to JPEGs for browsers like Firefox."
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/play-format-type.jpg"/>
</noscript>
</picture>
<figcaption>可以看到Google Play对Blink内核浏览器返回WebP的同时，对于像Firefox这样不支持WebP的浏览器返回JPEG。</figcaption>
</figure>


以下是从服务端返回WebP给你的用户的几个方案：

**使用 .htaccess配置来提供WebP的副本**

下面将说明当在服务器上存在与JPEG或PNG图像相匹配的.webp版本时，如何使用.htaccess文件将WebP文件提供给支持的浏览器。

Vincent Orback推荐使用如下方法：

浏览器可以通过Header中指定[Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept)来显式的[指定WebP支持](http://vincentorback.se/blog/using-webp-images-with-htaccess/)。通过这个标识，你就可以控制你的服务端，返回图像的WebP版本（如果这个WebP图像已经储存在磁盘上，而不是JPEG或PNG格式）。这个方法并不总是有效（例如，对于静态化的网站，如Github或S3）,所以在选择这种方案时请先检查下它对于你的网站是否有效。

以下是Apache Web服务器的.htaccess文件示例：

```
<IfModule mod_rewrite.c>

  RewriteEngine On

  # Check if browser support WebP images
  RewriteCond %{HTTP_ACCEPT} image/webp

  # Check if WebP replacement image exists
  RewriteCond %{DOCUMENT_ROOT}/$1.webp -f

  # Serve WebP image instead
  RewriteRule (.+)\.(jpe?g|png)$ $1.webp [T=image/webp,E=accept:1]

</IfModule>

<IfModule mod_headers.c>

    Header append Vary Accept env=REDIRECT_accept

</IfModule>

AddType  image/webp .webp
```

如果页面上显示的.webp图像有问题，那么请确保在服务器上启用了image/webp类型支持。开启方法如下：

Apache：将以下代码添加到.htaccess文件中：

```
AddType image/webp .webp
```

Nginx：将以下代码添加到您的mime.types文件中：

```
image/webp webp;
```

<aside class="note"><b>注意:</b> Vincent Orback提供了一个[htaccess配置](https://github.com/vincentorback/WebP-images-with-htaccess)示例，可以作为参考；Ilya Grigorik维护一组用于为WebP提供服务[配置脚本](https://github.com/igrigorik/webp-detect)。</aside>


**使用 `<picture>` 标签**

浏览器本身能够通过使用`<picture>`标签，来选择要显示的图像格式。`<picture>`标签利用多个`<source>`和一个`<img>`来设置显示的图像，其中`<img>`是包含了图像的、真实的DOM元素。浏览器会循环遍历`<source>`提供的链接，并检索到第一个匹配的结果。如果用户的浏览器中不支持`<picture>`，浏览器将会将它渲染为一个`<div>`，并使用其中的`<img>`。

<aside class="note"><b>注意:</b> 注意`<source>`的排列顺序。不要将image/webp的格式放在旧格式的后面，而是将它们放在前面。浏览器将会先解析到它们并得到匹配，而不会解析到其他支持更广泛的格式。如果物理尺寸相同（不使用`media`属性），也可以按照文件大小的顺序放置图像。一般来说，解析的顺序就是摆放的顺序。 </aside>

以下是一些HTML示例：

```html
<picture>
  <source srcset="/path/to/image.webp" type="image/webp">
  <img src="/path/to/image.jpg" alt="">
</picture>

<picture>   
    <source srcset='paul_irish.jxr' type='image/vnd.ms-photo'>  
    <source srcset='paul_irish.jp2' type='image/jp2'>
    <source srcset='paul_irish.webp' type='image/webp'>
    <img src='paul_irish.jpg' alt='paul'>
</picture>

<picture>
   <source srcset="photo.jxr" type="image/vnd.ms-photo">
   <source srcset="photo.jp2" type="image/jp2">
   <source srcset="photo.webp" type="image/webp">
   <img src="photo.jpg" alt="My beautiful face">
</picture>
```

**使用自动化的CDN转换WebP**

一些CDN支持将图像自动转换为WebP，可以使用[Client Hints](http://cloudinary.com/documentation/responsive_images#automating_responsive_images_with_client_hints)来让后端尽可能地提供WebP图像。请检查您的CDN，查看服务中是否包含了WebP支持。你可能有一个简单的解决方案，只是等待着你去寻找。

**WordPress的WebP支持**

**Jetpack** — Jetpack是一款流行的WordPress插件，它包括一个名为[Photon](https://jetpack.com/support/photon/)的CDN映像服务。使用Photon，你就可以获得无缝的WebP图像支持。这个叫Photon的CDN服务是包含在Jetpack的免费清单中的，因此这是一个很棒很有效的实现。不过，缺点是Photon会调整您的图像大小，将一个查询字符串放在您的URL中，并且每个图像都需要额外的DNS查找。

**Cache Enabler and Optimizer** — 如果您使用的是WordPress，那么你至少有一个半开源的选项。WordPress的开源插件[Cache Enabler](https://wordpress.org/plugins/cache-enabler/)有一个菜单复选框可以勾选，用于缓存WebP图像，如果当前用户的浏览器支持它就会提供给用户。这使得WebP图像服务变得容易。但是，这个插件有一个缺点：Cache Enabler需要使用一个名为Optimizer的姐妹程序，而它则是需要支付年费的。这对于一个需要真正的开源解决方案的情况来说，显然是不合适的。

**Short Pixel** — Cache Enabler的另一个搭配选择是使用Short Pixel，它的功能和Optimizer很像，也是付费的。但是，Short Pixel可以每个月免费优化100张图片。

**最好使用`<video>`压缩GIF动画**

动画GIF使用依然广泛，尽管这是一个功能非常有限的格式。虽然从社交网络到流行的媒体网站都大量嵌入了动画GIF，但是其实这个格式*从未*被设计用于视频或动画存储。事实上，[GIF89a规范](https://www.w3.org/Graphics/GIF/spec-gif89a.txt)一个在提示“GIF不是作为动画的平台”。格式中[颜色、图层和纬度的数量](http://gifbrewery.tumblr.com/post/39564982268/can-you-recommend-a-good-length-of-clip-to-keep-gifs)都影响GIF动画文件的体积，切换到视频格式可以提供最大的空间节省。


<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/animated-gif.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/animated-gif.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/animated-gif.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/animated-gif.jpg"
        alt="Animated GIF vs. Video: a comparison of file sizes at ~equivalent quality for different formats."
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/q_100/v1502426282/essential-image-optimization/animated-gif.jpg"/>
</noscript>
</picture>
<figcaption>动画GIF对比视频：同等质量下不同文件格式的文件大小对比</figcaption>
</figure>

**提供相同内容和质量的MP4文件可能会在文件体积上节省达到80％或更多。**GIF通常不仅会浪费大量的带宽，而且加载时间较长、只支持较少的颜色，因此只能使网站提供的用户体验大打折扣。您可能已经注意到，上传到Twitter的动画GIF在Twitter上的表现是要优于其他网站的。这是因为，[Twitter上的动画GIF实际上并不是GIF](http://mashable.com/2014/06/20/twitter-gifs-mp4/#fiiFE85eQZqW)。为了提高用户体验和减少带宽占用，上传到Twitter的动画GIF实际上是被转换成为了视频。同样，[Imgur](https://thenextweb.com/insider/2014/10/09/imgur-begins-converting-gif-uploads-mp4-videos-new-gifv-format/)也将上传的GIF转换为了静音的MP4视频。

为什么GIF多数情况下要大很多？因为，动画GIF将每个帧存储为了无损GIF图像 - 是的，无损。我们经常遇到的GIF的质量下降，其实是因为GIF受限于它的256色调色板。GIF格式下文件通常很大，因为它不像H.264这样的视频编解码器会考虑压缩相邻帧。MP4视频将每个关键帧存储为有损的JPEG，并其丢弃一些原始数据以实现更好的压缩。

**如果您可以切换到视频**

*   使用[ffmpeg](https://www.ffmpeg.org/)转换你的动画GIF（或原动画）为H.264 MP4视频。我一般是使用[Rigor](http://rigor.com/blog/2015/12/optimizing-animated-gifs-with-html5-video)提供的一行命令:

  ```
  ffmpeg -i animated.gif -movflags faststart -pix_fmt yuv420p -vf "scale=trunc(iw/2)*2:trunc(ih/2)*2" video.mp4
  ```

  ​

*   ImageOptim的API还支持[转换动画GIF为WebM/H.264视频](https://imageoptim.com/api/ungif)，另外还可以[从GIF中消除抖动](https://github.com/pornel/undither#examples)，从而帮助视频编解码器压缩视频到更小。

**如果您必须使用动画GIF**

*   可以使用像Gifsicle这样的工具，清除元数据和未使用的调色板条目，并最小化帧之间的变化。
*   考虑使用一个有损的GIF编码器。Gifsicle派生的[Giflossy](https://github.com/pornel/giflossy)支持一个`—lossy`命令标识，可以删除掉60%~65％文件体积。还有一个很好的基于它的工具，称为[Gifify](https://github.com/vvo/gifify)。对于非动画GIF，将其转换为PNG或WebP。

有关更多信息，请查阅Rigor编写的[关于GIF的电子书](https://rigor.com/wp-content/uploads/2017/03/TheBookofGIFPDF.pdf).

## <a id="svg-optimization" href="#svg-optimization">SVG的优化</a>

保持SVG的优良，意味着要清除任何不必要的东西。使用编辑器创建的SVG文件通常包含大量冗余信息（元数据、注释、隐藏层等）。通常可以安全地删除此内容，或将其转换为更小的形式，而不影响当前要显示的最终SVG。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image26.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image26.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image26.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image26.jpg"
        alt="svgo"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image26.jpg"/>
</noscript>
</picture>
<figcaption>由Jake Archibald提供的[SVGOMG](https://jakearchibald.github.io/svgomg/)，提供了一个GUI界面方便你通过选择选项优化你的SVG，并支持实时预览。 </figcaption>
</figure>

**SVG优化的一些通用规则（SVGO）：**

*   使用Minify和gzip压缩您的SVG文件。SVG实际上只是以XML格式表示的文本资源，和CSS、HTML以及JavaScript是一样的，我们应该使用Minify和gzip压缩它以提高性能。
*   使用预定义的SVG图形如`<rect>`，`<circle>`，`<ellipse>`，`<line>`和`<polygon>`取代路径。优选预定义的形状有助于减少生成最终图像所需的标签量，也意味着较少的浏览器解析和点阵描述代码。减少SVG复杂度也意味着浏览器可以更快地显示它。
*   如果您必须使用路径（Path），请尝试减少曲线路径，尽量简化和合并它们。Illustrator的[简化工具](http://jlwagner.net/talks/these-images/#/2/10)可以帮助您在复杂的艺术品中消除多余的点，同时平滑不规则的曲线。
*   避免使用组（Group）。如果不能，请尝试简化它们。
*   删除不可见的图层。
*   避免使用任何Photoshop或Illustrator效果。它们会使生成较大的位图图像。
*   仔细检查SVG中任何非友好的嵌入的位图图像。
*   使用工具优化SVG。 [SVGOMG](https://jakearchibald.github.io/svgomg/)是一个Jake Archibald为[SVGO](https://github.com/svg/svgo)编写的一个方便的Web端操作界面。如果你使用Sketch，可以在导出时使用[SVGO压缩插件]([Sketch plugin for running SVGO](https://www.sketchapp.com/extensions/plugins/svgo-compressor/))以减小导出文件的体积。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/svgo-precision.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/svgo-precision.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/svgo-precision.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/svgo-precision.jpg"
        alt="svgo precision reduction can sometimes have a positive impact on size"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/svgo-precision.jpg"/>
</noscript>
</picture>
<figcaption>使用SVGO高精度模式（体积减少29%）和低精度模式（体积减少38%）处理SVG原图后的对比示例。</figcaption>
</figure>


[SVGO](https://github.com/svg/svgo)是一种Node.js环境下优化SVG的工具。SVGO可以通过减少你的路径（Path）中的精度点数，来减少最终文件的体积。每增加一个点位数后就会增加一个字节，这就是为什么更改精度（位数）会严重影响文件的体积。但是，改变精度需要非常小心，因为它会影响你的图形的视觉效果。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image28.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image28.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image28.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image28.jpg"
        alt="where svgo can go wrong, oversimplifying paths and artwork"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image28.jpg"/>
</noscript>
</picture>
<figcaption>需要注意的是，虽然SVGO在前面的例子中都表现良好，并不会过分简化曲线和形状，但是有很多情况下可能不是这样。如上图，观察火箭上的线条可以看到在较低的经度下，线条是如何产生了变形。</figcaption>
</figure>

**在命令行中使用SVGO：**

如果您更喜欢GUI，SVGO可以作为[全局的npm CLI](https://www.npmjs.com/package/svgo)安装：

```
npm i -g svgo
```

然后可以对本地的SVG文件执行，如下所示：

```
svgo input.svg -o output.svg
```

它支持您可能期望的所有选项，包括调整浮点精度：

```
svgo input.svg --precision=1 -o output.svg
```

有关支持选项的完整列表，请参阅SVGO [说明文件](https://github.com/svg/svgo)。

**不要忘了压缩SVG！**

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/before-after-svgo.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/before-after-svgo.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/before-after-svgo.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/before-after-svgo.jpg"
        alt="before and after running an image through svgo"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/before-after-svgo.jpg"/>
</noscript>
</picture>
</figure>

此外，不要忘记[使用Gzip压缩你的SVG资源](https://calendar.perfplanet.com/2014/tips-for-optimising-svg-delivery-for-the-web/)或者使用Brotli来提供服务。因为SVG是文本的，所以压缩率会非常高（可以减少50%）。

当Google发布了一个新徽标时，我们宣布其[最小](https://twitter.com/addyosmani/status/638753485555671040)版本的大小只有305个字节。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image30.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image30.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image30.jpg" />

<img
        class="lazyload very-small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image30.jpg"
        alt="the smallest version of the new google logo was only 305 bytes in size"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image30.jpg"/>
</noscript>
</picture>
</figure>

有[很多高级的SVG技巧](https://www.clicktorelease.com/blog/svg-google-logo-in-305-bytes/)可以用来将其进一步删减体积（一直到146个字节）！可以说，无论是通过工具还是手动清理，可能你都有可能将SVG的体积再刮掉一些。

**SVG Sprite**

SVG在制作图标时非常[强大](https://css-tricks.com/icon-fonts-vs-svg/)，它就像一个精灵一样，提供了一种表示可视化图形的方式，在这种方式里没有[奇怪](https://www.filamentgroup.com/lab/bulletproof_icon_fonts.html)的必须使用的字体。它有着比图标字体（SVG笔触属性）更精准的CSS样式控制，更好的定位控制（不需要各种伪元素和CSS `display`），并且SVG 更容易[理解](http://www.sitepoint.com/tips-accessible-svg/)。

像[svg-sprite](https://github.com/jkphl/svg-sprite)和[IcoMoon](https://icomoon.io/)这样的工具，可以自动将[SVG](https://github.com/jkphl/svg-sprite)组合成sprite，并可以通过[CSS Sprite](https://css-tricks.com/css-sprites/)，[Symbol Sprite](https://css-tricks.com/svg-use-with-external-reference-take-2)或[Stacked Sprite](http://simurai.com/blog/2012/04/02/svg-stacks)来使用。Una Kravetz有一篇实用的[文章](https://una.im/svg-icons/#💁)值得看一下，其中说明了如何使用gulp-svg-sprite进行SVG Sprite工作流程。Sara Soudein也曾在她的博客中表述[转变图标字体到SVG](https://www.sarasoueidan.com/blog/icon-fonts-to-svg/)。

**进一步阅读**

Sara Soueidan的[网页交付中的优化SVG技巧](https://calendar.perfplanet.com/2014/tips-for-optimising-svg-delivery-for-the-web/)和Chris Coyier的[实用SVG](https://abookapart.com/products/practical-svg)电子书都非常好。我还发现Andreas Larsen的优化SVG帖子很有启发（[第1 ](https://medium.com/larsenwork-andreas-larsen/optimising-svgs-for-web-use-part-1-67e8f2d4035)[部分](https://medium.com/larsenwork-andreas-larsen/optimising-svgs-for-web-use-part-2-6711cc15df46)，[第2部分](https://medium.com/larsenwork-andreas-larsen/optimising-svgs-for-web-use-part-2-6711cc15df46)）。另外，[在Sketch中准备和导出SVG图标](https://medium.com/sketch-app-sources/preparing-and-exporting-svg-icons-in-sketch-1a3d65b239bb)也是一个很好的借鉴。

## <a id="avoid-recompressing-images-lossy-codecs" href="#avoid-recompressing-images-lossy-codecs">[避免使用有损编解码器重复压缩图像](https://images.guide/#avoid-recompressing-images-lossy-codecs)</a>

建议始终从原始图像开始压缩。重复压缩图像会带来恶果。假设您拍摄的JPEG已经被压缩，质量为60。如果再用有损编码重新压缩此图像，那它看起来会更糟。每一轮的压缩都会带来代数损失 - 信息将会丢失，并且开始产生虚影。即使您在高质量的环境下，重复压缩依然会是不好的结果。

为了避免这个陷阱，首先你要**设置愿意接受的最低质量值**，这样你从一开始就可以节省最多的文件存储空间。同时，您还可以避免这种陷阱，因为任何文件体积的降低必然带来可视质量的降低。

重新编码有损文件几乎总是给你一个更小的文件，但这并不意味着你可以像你所想的那样获得尽可能好的质量。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/generational-loss.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/generational-loss.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/generational-loss.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/generational-loss.jpg"
        alt="generational loss when re-encoding an image multiple times"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/generational-loss.jpg"/>
</noscript>
</picture>
<figcaption>以上，从Jon Sneyers的[优秀视频](https://www.youtube.com/watch?v=w7vXJbLhTyI)和[随附文章](http://cloudinary.com/blog/why_jpeg_is_like_a_photocopier)中，我们可以看到多种图像格式被重新压缩后的影响。如果你从社交网络中保存（已压缩）图像，并重新上传（导致重新压缩），就会遇到这个问题：质量损失将会急剧增加。</figcaption>
</figure>


因为网格量化的原理，MozJPEG（或许不小心）具有了更好的抗再压缩性能。不同于精确的压缩所有的DCT值，MozJPEG会检查像素中+1/-1范围内的近似值，以查看相似的值是否压缩到较少的字节。有损的FLIF在（重新）压缩之前会有一个类似于有损PNG的黑科技，它可以查看文件的数据并决定丢弃什么。重新压缩的PNG它具有可以检测的“空洞”，以避免进一步的丢弃数据。

**因此，编辑源文件时，请将其存储为无损格式（如PNG或TIFF），以便尽可能保留质量。**这样，您的构建工具或图像压缩服务导出的压缩图像，才能在提供给你的用户时保持最小的质量损失。

## <a id="reduce-unnecessary-image-decode-costs" href="#reduce-unnecessary-image-decode-costs">[减少不必要的图像解码和尺寸调整带来的损耗](https://images.guide/#reduce-unnecessary-image-decode-costs)</a>

我们以前都尽量提供体积大的、高清晰的图像给用户，甚至根本就超过了用户的需求。这样做是有代价的。解码和调整图像尺寸，对于大多数的移动设备上的浏览器来说，是个昂贵的操作。如果发送大图像，再使用CSS或width/height属性重新调整尺寸，您可能会经历下面的等待，这是非常影响设备性能的。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1503503389/essential-image-optimization/image-pipeline.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1503503389/essential-image-optimization/image-pipeline.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503503389/essential-image-optimization/image-pipeline.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503503389/essential-image-optimization/image-pipeline.jpg"
        alt="There are many steps involved in a browser grabbing an image specified in a tag and displaying it on a screen. These include request, decode, resize, copy to GPU and display."
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503503389/essential-image-optimization/image-pipeline.jpg"/>
</noscript>
</picture>
<figcaption>当浏览器获取图像的时候，它必须将图像从原始源格式（例如JPEG）解码为内存中的位图。而通常，图像又需要调整尺寸（例如，宽度已设置为其容器的百分比）。图像的解码和调整尺寸的操作是昂贵的，将会很大程度上延迟图片显示的时间。 </figcaption>
</figure>

发送给浏览器可以渲染的图像，并且不需要调整尺寸，才是最理想的情况。所以，要为你的用户提供适合屏幕尺寸和分辨率的最小图像，利用`<img>`的[`srcset`和`sizes`属性](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images) - 我们将很快的覆盖`srcset`，快速显示出图像。

忽略图像（`<img>`）上的属性`width`或`height`属性也可能会对性能产生负面影响。因为没有它们的话，浏览器会先为图像分配一个较小的占位符区域，然后直到足够的字节到达才能知道正确的尺寸。这种处理方式下，页面的文档布局就必须要更新，而更新可能会是一个昂贵的步骤，它被称做回流。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/devtools-decode.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/devtools-decode.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/devtools-decode.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/devtools-decode.jpg"
        alt="image decode costs shown in the chrome devtools"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/devtools-decode.jpg"/>
</noscript>
</picture>
<figcaption>浏览器必须经过多个步骤才能在屏幕上绘制图像。除了要获取图像，还需要对图像进行解码而且经常需要调整尺寸。这些事件可以通过Chrome开发者工具中的Performance面板中的[时间线](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/performance-reference)进行查看审核。 </figcaption>
</figure>

当然，更大的图像也会带来更大的内存占用。解码图像每个像素需要占用4个字节。如果你不小心，你可以真正地让浏览器崩溃; 在一些低端设备上，并没有多大的空间来进行内存交换。所以，请一定要注意您的图像解码、尺寸调整和内存占用带来的影响。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1503695136/essential-image-optimization/image-decoding-mobile.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1503695136/essential-image-optimization/image-decoding-mobile.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503695136/essential-image-optimization/image-decoding-mobile.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503695136/essential-image-optimization/image-decoding-mobile.jpg"
        alt="Decoding images can be incredibly costly on average and lower-end mobile hardware"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503695136/essential-image-optimization/image-decoding-mobile.jpg"/>
</noscript>
</picture>
<figcaption>解码图像的成本在中低端的手机上，可能是非常昂贵的。在某些情况下，它的速度可能要慢上5倍还多。</figcaption>
</figure>

Twitter（推特）在构建它们新的[移动网络体验](https://medium.com/@paularmstrong/twitter-lite-and-high-performance-react-progressive-web-apps-at-scale-d28a00e780a3)时，通过确保向用户提供最佳适配尺寸的图像来提高图像解码的性能。这使得Twitter的许多图像在时间轴上测试的解码时间从400ms一直下降到19ms！

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/image-decoding.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/image-decoding.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/image-decoding.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/image-decoding.jpg"
        alt="Chrome DevTools Timeline/Performance panel highlighting image decode times before and after Twitter Lite optimized their image pipeline. Before was higher."
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/image-decoding.jpg"/>
</noscript>
</picture>
<figcaption>Chrome开发者工具中的时间轴/性能面板上，对比Twitter优化其图像管道之前和之后显示的图像解码时间（绿色）。/figcaption>
</figure>

### <a id="delivering-hidpi-with-srcset" href="#delivering-hidpi-with-srcset">使用`srcset`提供HiDPI图像</a>

用户可以通过各种具有高分辨率屏幕的移动设备和桌面设备来访问您的站点。在Web页面中，[设备像素比](https://stackoverflow.com/a/21413366)（DPR）（也称为“CSS像素比”）被用来确定一个设备的屏幕分辨率如何被CSS来解释。DPR是由电话制造商创建的，被用于在提高移动屏幕的分辨率和清晰度的情况下，不会使显示的元素感觉太小。

而为了适配到用户可能期望的图像质量，需要将最合适的分辨率的图像提供到用户的设备上。高DPR图像（例如2x，3x）被聪明地提供给支持它们的设备，而较低或标准的DPR图像则被提供给那些没有高分辨率屏幕的用户，因为这些所谓的2x+的图像通常会显著地占用更多的字节空间。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502570356/essential-image-optimization/device-pixel-ratio.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502570356/essential-image-optimization/device-pixel-ratio.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502570356/essential-image-optimization/device-pixel-ratio.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502570356/essential-image-optimization/device-pixel-ratio.jpg"
        alt="A diagram of the device pixel ratio at 1x, 2x and 3x. Image quality appears to sharpen
        as DPR increases and a visual is shown comparing device pixels to CSS pixels."
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502570356/essential-image-optimization/device-pixel-ratio.jpg"/>
</noscript>
</picture>
<figcaption>设备像素比：许多网站可以检测流行设备的DPR，例如[material.io](https://material.io/devices/)和[mydevice.io](https://mydevice.io/devices/)。</figcaption>
</figure>


[srcset](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)属性允许浏览器为每个设备选择最佳可用的图像，例如为2x移动设备显示2x图像。没有`srcset`支持的浏览器则会使用`<img>`标签中`src`指定的默认值。

```
<img srcset="paul-irish-320w.jpg,
             paul-irish-640w.jpg 2x,
             paul-irish-960w.jpg 3x"
     src="paul-irish-960w.jpg" alt="Paul Irish cameo">
```

像[Cloud](http://cloudinary.com/blog/how_to_automatically_adapt_website_images_to_retina_and_hidpi_devices)和[Imgix](https://docs.imgix.com/apis/url/dpr)这样的图像CDN服务都支持控制图像密度，并支持从单一的规范来源为用户提供最佳密度的图像。

<aside class="note"><b>注意:</b> 您可以通过这个免费的[Udacity（优达学城）](https://www.udacity.com/course/responsive-images--ud882)课程和Web基础知识中的[图像指南](https://developers.google.com/web/fundamentals/design-and-ui/responsive/images)，了解有关设备像素比和响应图像的更多信息。</aside>

友好的提醒你，[Client Hints](https://www.smashingmagazine.com/2016/01/leaner-responsive-images-client-hints/)也提供了一个替代的标记方案来指定每个可能的像素密度和图像格式，借此来实现响应图像。不同的是，此信息是被附加到HTTP请求中，而Web服务器就可以通过它来选择最适合当前设备屏幕密度的图像来返回回去。

### <a id="art-direction" href="#art-direction">艺术化的响应</a>

尽管向用户提供适配分辨率的内容很重要，但是有些网站可能会需要考虑艺术化的响应图像像是。如果用户位于一个较小的屏幕上，则可能需要裁剪或放大图像，以充分利用可用的空间突出图像的主题内容。虽然美术设计方面的内容超出了这个写作的范围，但像[Cloudinary](http://cloudinary.com/blog/automatically_art_directed_responsive_images%20)的服务是提供了这样的API，来尽可能地尝试自动化的处理类似的需求。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/responsive-art-direction.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/responsive-art-direction.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/responsive-art-direction.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/responsive-art-direction.jpg"
        alt="responsive art direction in action, adapting to show more or less of an image in a cropped manner depending on device"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/responsive-art-direction.jpg"/>
</noscript>
</picture>
<figcaption>艺术化响应：Eric Portis放置了一个精彩的[样品](https://ericportis.com/etc/cloudinary/)来展示如何从美术设计方面响应图片。这个例子中通过焦点变换突出展示了焦点图片的主题内容，并充分利用了有限空间。.</figcaption>
</figure>

## <a id="color-management" href="#color-management">颜色管理</a>

世界上，至少有三种不同的颜色理论，来自不同的学科：生物学、物理学和印刷学。在生物学中，颜色是一种[感知现象](http://hubel.med.harvard.edu/book/ch8.pdf)。物体以不同的波长组合反射光，光传播到我们的眼中，眼中的光受体将这些波长转化成我们所知道的颜色。而在物理学中，重要的是光线 - 亮度和频率。在印刷学中，更多是关于色轮、油墨以及艺术模型。

在理想情况下，世界各地的屏幕和网页浏览器都应该显示出完全相同的颜色。可不幸的是，由于一些内在技术的不一致，他们并没有显示的完全相同。因此，我们需要颜色管理，它能够使我们通过颜色模型、空间和配置文件等手段来达到颜色的折中统一。

#### 颜色模型

[颜色模型](https://en.wikipedia.org/wiki/Gamma_correction)是用于从较小的原色组中派生出完整范围颜色的系统。其中有不同类型的颜色空间，它们使用不同的参数来控制颜色。有一些颜色空间的控制参数比其他颜色空间要更少 - 例如灰度只有一个参数，用于控制黑白颜色之间的亮度。 

两种常见的颜色模型是加色模型和减色模型。加色模型（如用于数字显示的RGB）使用亮色调来显示颜色（由黑至白），而减色模式（如打印中使用的CMYK）则是通过削弱白色来工作（由白至黑）。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1504564914/colors_ept6f2.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1504564914/colors_ept6f2.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504564914/colors_ept6f2.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504564914/colors_ept6f2.jpg"
        alt="sRGB, Adobe RGB and ProPhoto RGB" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1504564914/colors_ept6f2.jpg"/>
</noscript>
</picture>
<figcaption>在RGB中，红色、绿色和蓝色以不同的组合方式产生更多的颜色。而CYMK则是通过不同颜色（青色、品红色、黄色和黑色）的墨水组合在白纸上，使之减去亮度。 </figcaption>
</figure>

这篇文章《[了解颜色模型和专色系统](https://www.designersinsights.com/designer-resources/understanding-color-models/)》，可以帮助你了解其他颜色模型和模式（如HSL，HSV和LAB）。

#### 颜色空间

[颜色空间](http://www.dpbestflow.org/color/color-space-and-color-profiles#space)是一组特定的颜色范围，它限定了表示图像的颜色范围。例如，如果图像包含多达1670万种颜色，则不同的颜色空间允许使用这些颜色的范围会有所不同，可能更宽或更窄。一些开发者会认为颜色模型和颜色空间是相同的东西。

[sRGB](https://en.wikipedia.org/wiki/SRGB)被设计为网络的[标准](https://www.w3.org/Graphics/Color/sRGB.html)颜色空间。它是基于RGB的，是一个比较小的颜色空间。sRGB通常被认为是通用颜色的最小区域，是颜色管理中支持跨浏览器里最安全的选择。其他的颜色空间（如[Adobe RGB](https://en.wikipedia.org/wiki/Adobe_RGB_color_space)或[ProPhoto RGB](https://en.wikipedia.org/wiki/ProPhoto_RGB_color_space) - 在Photoshop和Lightroom中使用）其实可以表现出比sRGB更鲜艳的色彩，但后者（sRGB）在大多数网络浏览器、游戏和显示器上都是普遍存在的，因此受到普遍的关注。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1504565044/color-wheel_hazsbk.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1504565044/color-wheel_hazsbk.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504565044/color-wheel_hazsbk.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504565044/color-wheel_hazsbk.jpg"
        alt="sRGB, Adobe RGB and ProPhoto RGB" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1504565044/color-wheel_hazsbk.jpg"/>
</noscript>
</picture>
<figcaption>上图我们可以看到色域的可视化范围 - 即颜色空间定义的颜色范围。</figcaption>
</figure>

颜色空间有三个通道（红色、绿色和蓝色）。在8位模式下，每个通道有255种颜色可供使用，共有1670万种颜色。而16位模式下，图像可以显示数万亿种颜色。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1504564915/srgb-rgb_ntuhi4.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1504564915/srgb-rgb_ntuhi4.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504564915/srgb-rgb_ntuhi4.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504564915/srgb-rgb_ntuhi4.jpg"
        alt="sRGB, Adobe RGB and ProPhoto RGB" />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1504564915/srgb-rgb_ntuhi4.jpg"/>
</noscript>
</picture>
<figcaption>来自[Yardstick](https://yardstick.pictures/tags/img%3Adci-p3)的sRGB、Adobe RGB和ProPhoto RGB颜色空间下的图片对比。如果无法向你展示出那些你无法看到的颜色，那么想在sRGB空间下解释它们的不同是非常困难的（，因为你的浏览器应该也是使用的sRGB颜色空间）。（所以，在这种情况下，）宽色域和sRGB下的常规照片应该是完全一样的，除了那些最饱和的“刺激性的”颜色。</figcaption>
</figure>

颜色空间（如sRGB，Adobe RGB和ProPhoto RGB）之间的差异来自于它们的色域（可用色调再现的颜色范围）、光源以及[伽玛](http://blog.johnnovak.net/2016/09/21/what-every-coder-should-know-about-gamma/)曲线的不同。sRGB的颜色范围比Adobe RGB小约20％，而ProPhoto RGB要比Adobe RGB 大约[50％](http://www.petrvodnakphotography.com/Articles/ColorSpace.htm)。上面的图像来源于[Clipping Path](http://clippingpathzone.com/blog/essential-photoshop-color-settings-for-photographers)。

[宽色域](http://www.astramael.com/)是一个用于描述颜色空间的术语，其色域范围大于sRGB。目前，这种类型的显示器正在变得越来越普遍。同时也表明，仍然有许多的数字显示器无法显示比sRGB明显更好的颜色配置文件。所以，在Photoshop中使用“Saving for Web”时，请还是考虑使用“转换为sRGB”的选项，除非你确定你的图像是定位提供给具有较高端宽色域屏幕的用户。

<aside class="key-point"><b>注意:</b> 当进行最初的摄影时，请避免使用sRGB作为主要的颜色空间。因为它比大多数相机支持的颜色空间要小，所以会导致原始照片被剪辑。取而代之，应该在色域更广的颜色空间（如ProPhoto RGB）上拍摄照片，在导出的Web使用时再输出到sRGB颜色空间。∂</aside>

**有没有什么方法让Web中的图像看起来颜色更饱和？**

有。你的图片可能包含了一些饱和度高、有冲击力或十分艳丽的颜色，所以你会很关心它在屏幕上是否依然一样的鲜艳饱和。然而，这很少会出现在真实的照片上。通常情况下，可以很容易的调整一下颜色让它显示起来更加鲜艳，而不必超出sRGB的色域范围。

那是因为，人的色彩感觉并不是绝对的，而是相对于我们的周围环境，所以它很容易被欺骗。如果你的图像中含有类似荧光笔一样的高亮颜色，那么你的图像会感觉拥有了更广的色域。

#### 伽玛校正和压缩

[伽玛校正](https://en.wikipedia.org/wiki/Gamma_correction)（或只是伽玛）被用于控制一个图像的整体亮度。通过改变伽玛，也可以改变红色、绿色与蓝色的比例。一个图像如果没有进行伽马校正，它们的颜色很可能看起来像被漂白了或者会很暗淡。

在视频和计算机图形学领域中，伽马被用于压缩，类似于数据压缩。它允许你以较少的位数（8位而不是12或16）来压缩显示有用的亮度级别。人类对亮度的感知与光的物理量是不成线性比例。所以，在为人类的眼睛编码图像时，如果以真实物理形式表示颜色将是很浪费的。所说的伽玛压缩，就是在更接近人类感知的尺度上对亮度进行编码。

使用伽玛压缩比较有用的亮度级别，是使用8位精度（大多数RGB颜色使用0-255）。这其中的原理来自于：假设将颜色使用与物理学有1：1关系的一些单位来表示，在RGB值从1到百万的过程中，其中从0到1000将会看起来明显不同，而在999000到1000000之间的值变化将几乎是相同的。可以想象一下，在一个只有一个蜡烛的黑暗的房间里，点亮第二个蜡烛，你会明显注意到室内的光线亮度增加；而添加第三个蜡烛，房间看起来也会更加明亮；但是，想象在一个100蜡烛的房间里，点燃第101支蜡烛、第102只，你几乎完全注意不到亮度的变化。

即使在这两种情况下，从物理角度上说，它们都添加了完全相同的光量。因此基于此种眼睛的感知方式，在光线较弱时，伽马压缩会“压缩”这些亮度值，即使在物理学上亮度值会不太精确，但从人眼的角度来看，它的尺度会被眼睛调整，所有的值都是同样精确的。

<aside class="key-point"><b>注意:</b> 这里的伽玛压缩/修正与你可能在Photoshop中配置的图像的伽玛曲线是有所不同的。当对图片进行伽玛压缩后，你应该感觉不出来它有什么变化。</aside>

#### 颜色配置文件

颜色配置文件是当前设备的颜色空间的描述信息。它被用于在不同的颜色空间之间进行转换。配置文件被用于尝试确保图像在这些不同类型的屏幕和介质上，看起来尽可能的相似。

图像可以包含一个由[国际色彩联盟](http://www.color.org/icc_specs2.xalter)（ICC）所描述的嵌入式颜色配置文件，以准确表示颜色应如何显示。这是由不同的格式支持的，包括了JPEG，PNG，SVG和[WebP](https://developers.google.com/speed/webp/docs/riff_container)，大多数主流浏览器都支持嵌入式ICC配置文件。当在应用程序中显示图像并且它获取到显示器的功能时，就可以根据颜色配置文件调整这些颜色。

<aside class="key-point"><b>注意:</b>一些显示器具有类似于sRGB的颜色配置文件，并且根本不支持使用更好的颜色配置文件，因此嵌入配置文件的价值可能很有限。所以，请先检查你的目标显示器是什么样的。</aside>

嵌入式的颜色配置文件也会大大增加图像的文件体积（100KB+偶尔），所以还是要小心嵌入。像ImageOptim这样的工具实际上是会[自动](https://imageoptim.com/color-profiles.html)删除嵌入式的颜色配置文件的。事实上，以体积缩小为名删除ICC配置文件的话，浏览器将会被迫在显示器的颜色空间中显示图像，这可能导致无法显示预期的饱和度和对比度。所以，根据你的用例权衡利弊是有必要的。

如果您有兴趣了解ICC配置文件的更多信息，Nine Degrees在[这里](https://ninedegreesbelow.com/photography/articles.html)提供了一系列关于ICC配置文件颜色管理的资源。

#### 颜色配置文件和Web浏览器

在Chrome的早期版本，对颜色管理并没有很大的支持。但是在2017年，Chrome正在改善对[颜色的正确渲染](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/ptuKdRQwPAo)。对于不是sRGB（较新的MacBook Pros）的显示器，将会将颜色从sRGB转换为正确的显示配置文件。这意味着不同系统和浏览器上的颜色应该看起来更相似。Safari，Edge和Firefox现在也正在考虑支持ICC配置文件，因此，具有不同颜色配置文件（例如ICC）的图像现在已经可以正确的显示它们了，无论您的屏幕是否具有宽色域。

<aside class="key-point"><b>注意:</b> 对于如何让颜色在广色域下工作于Web环境的方式，Sarah Drasner提供了个更好的指南，请参阅[网页颜色的傻瓜式手册](https://css-tricks.com/nerds-guide-color-web/).</aside>

## <a id="image-sprites" href="#image-sprites">图像拼合技术</a>

[图像拼合技术](https://developers.google.com/web/fundamentals/design-and-ui/responsive/images#use_image_sprites) （或者说CSS图像拼合技术）在Web开发上其实有着悠久的历史，它受到所有浏览器的支持，这个技术是通过加载单个更大的合并图像来减少页面加载图像数量的一种流行方式。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1503693437/essential-image-optimization/i2_2ec824b0_1.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1503693437/essential-image-optimization/i2_2ec824b0_1.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503693437/essential-image-optimization/i2_2ec824b0_1.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503693437/essential-image-optimization/i2_2ec824b0_1.jpg"
        alt="Image sprites are still widely used in large, production sites, including the Google homepage."
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503693437/essential-image-optimization/i2_2ec824b0_1.jpg"/>
</noscript>
</picture>
<figcaption>图像拼合仍然广泛应用于很多大型网站的生产环境中，包括Google主页。</figcaption>
</figure>

在HTTP/1.x下，一些开发人员通过合并来减少加载图像的HTTP请求数量。这带来了一些好处，但是它很快遇到了“缓存无效”的挑战，需要小心的是 - 对于图像的任何的一小部分的更改，都将使得用户的缓存中的整个图像无效。

现在，拼合技术在[HTTP/2](https://hpbn.co/http2/)可能是不合适的。使用HTTP/2的话，最好是[加载单个图像](https://deliciousbrains.com/performance-best-practices-http2/)，因为现在单个链接中的多个请求是可能的。因此，使用之前请评估一下，图像拼合技术是否适用于你的网络设置。

## <a id="lazy-load-non-critical-images" href="#lazy-load-non-critical-images">[延迟加载非关键图像](https://images.guide/#lazy-load-non-critical-images)</a>

延迟加载是一种优化Web性能的模式，它会延迟浏览器中图像的加载，直到用户需要看到它的时候。一个例子是，当您滚动页面时，可视区域的图像会按需异步加载。这可以进一步帮助你节省从图片压缩策略省出来的带宽空间。


<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/scrolling-viewport.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/scrolling-viewport.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/scrolling-viewport.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/scrolling-viewport.jpg"
        alt="lazy-loading images"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/scrolling-viewport.jpg"/>
</noscript>
</picture>
</figure>

出现在“折叠线之上”或网页中首次出现的图像必须立即加载。然而，在“折叠线下方”的图像对于用户来说尚不可见，它们不必被立即加载到浏览器中。只有当用户向下滚动并且需要显示它们时，再稍后加载或者延迟加载即可。

其实，浏览器本身并没有支持延迟加载（尽管过去曾经[讨论](https://discourse.wicg.io/t/a-standard-way-to-lazy-load-images/1153/10)过）。因此，我们会使用JavaScript来添加此功能。

**为什么延迟加载是有效的？**

这种所谓“延迟”，即只有在必要时加载图像的方式，有很多好处：

* **降低数据消耗**：由于你假设用户不需要提前提取每个图像，因此您只需加载最少的资源。这永远是一件好事，特别是在具有更严格的流量控制的移动设备上。
* **降低电池消耗**：减少了用户浏览器的工作量，也可以节省电池的使用寿命。
* **提高下载速度**：将图像占比较大的网站的整体页面加载时间，从几秒缩短到几乎没有，这是对用户体验的巨大提升。事实上，这可能是决定你的用户在你的网站长期驻留还是昙花一现的关键点。

**但是，就像所有的工具一样，强大的力量意味着巨大的责任**。**

**避免在折叠线之上放置图像。**延迟加载可以用于长列表的图像（例如产品）或用户头像列表中。但不要用于主页的焦点图像。延迟加载折叠线以上的图像将会使整个页面的加载显得很慢，无论是从技术上还是人类的感知上讲。我们可以使用JavaScript屏蔽浏览器的预加载、增加逐步加载，为浏览器创建额外的工作。

**在滚动时延迟加载图像，要非常谨慎。**如果你等待用户滚动之后再加载图像，他们可能就会看到图像占位符，并且很有可能在它们已经滚动过去后图像才加载完成。一个建议就是在加载折叠线之上的图像后就开始延迟加载，加载所有图像，而不依赖于用户的操作。

**谁在使用延迟加载技术？**

有关延迟加载的示例，请查看以托管大量图像为主营业务的所有站点。比较著名的网站是[Medium](https://medium.com/)和[Pinterest](https://www.pinterest.com/)。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image35.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image35.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image35.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image35.jpg"
        alt="inline previews for images on medium.com"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image35.jpg"/>
</noscript>
</picture>
<figcaption>在Medium.com上，一个使用高斯模糊在线预览的例子。</figcaption>
</figure>

一些网站（例如“Medium”）会先显示一个小的高斯模糊的在线预览图像（几百个字节），一旦获取完成，它将转换（延迟加载）到全质量的图像。

JoséM.Pérez曾经写过关于如何使用[CSS过滤器](https://jmperezperez.com/medium-image-progressive-loading-placeholder/)实现Medium网站的类似效果，并尝试了[不同的图像格式](https://jmperezperez.com/webp-placeholder-images/)来支持这样的占位符。Facebook也曾为其著名的200字节方式占位符，撰写了一份值得一读的文章“[封面照片](https://code.facebook.com/posts/991252547593574/the-technology-behind-preview-photos/)”。如果你是Webpack的使用者，[LQIP加载程序](https://lqip-loader.firebaseapp.com/)可以帮助你自动完成类似的工作。

事实上，您可以搜索你最喜欢的高清照片来源，然后向下滚动页面。几乎所有情况下，你都将体验到网站一次只能加载几个全分辨率的图像，其余的则是占位符颜色或图像。当你继续滚动时，占位符图像将被替换为全分辨率图像。这就是典型的延迟加载效果。

**如何在我的页面使用延迟加载？**

有一些技术和插件可支持延迟加载。我推荐Alexander Farkas编写的[lazysizes](https://github.com/aFarkas/lazysizes)，因为它的性能良好、功能齐全、并可选择通过浏览器的[“交叉观察者”API（IntersectionObserver API）](https://developers.google.com/web/updates/2016/04/intersectionobserver)集成，还支持插件扩展。

**我可以怎样使用Lazysizes？**

Lazysizes是一个JavaScript的库。它不需要配置，只需下载压缩后的js文件并将其包含在您的网页中即可使用。


这是从Lazysizes的README文件中获取的一些示例代码：

将CSS类“lazyload”添加到你的包含data-src或data-srcset属性的`<img>`或`<iframe>`标签上。

或者，您也可以添加指向低质量图像的src属性：

```html
<!-- 非响应式示例: -->
<img data-src="image.jpg" class="lazyload" />

<!-- 响应式示例，包括data-sizes为auto: -->
<img
    data-sizes="auto"
    data-src="image2.jpg"
    data-srcset="image1.jpg 300w,
    image2.jpg 600w,
    image3.jpg 900w" class="lazyload" />

<!-- iframe示例 -->

<iframe frameborder="0"
    class="lazyload"
    allowfullscreen=""
    data-src="//www.youtube.com/embed/ZfV-aYdU4uE">
</iframe>
```

实际上，在这本书的网络版本中，我将Lazysizes（尽管可以使用任何替代方案）与Cloudinary配合响应式地返回需求的图像。这允许我自由地尝试不同的尺度、质量、格式的值，而无需在响应式处理上付出很大的精力：

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502501095/essential-image-optimization/cloudinary-responsive-images.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502501095/essential-image-optimization/cloudinary-responsive-images.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502501095/essential-image-optimization/cloudinary-responsive-images.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502501095/essential-image-optimization/cloudinary-responsive-images.jpg"
        alt="Cloudinary supports on-demand control of image quality, format and several other features."
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502501095/essential-image-optimization/cloudinary-responsive-images.jpg"/>
</noscript>
</picture>
</figure>

**Lazysizes的功能包括：**

* 自动检测当前和未来的“lazyload”元素的可见性变化。
* 包括标准的响应图像支持（包括picture和srcset）。
* 添加多媒体列表功能的自动尺寸计算和别名。
* 可被用在CSS和JS使用比重较大的网页或网络应用程序上，处理数以百计的images或 iframe标签。
* 可扩展：支持插件
* 轻量级但成熟的延迟加载解决方案
* SEO已改进：不能在爬虫抓取时隐藏图像或资源。

**其他延迟加载的可选项**

Lazysizes并不是你唯一的选择。这里还有其他的延迟加载库：

*   [Lazy Load XT](http://ressio.github.io/lazy-load-xt/)
*   [BLazy.js](https://github.com/dinbror/blazy) (或[Be]Lazy)
*   [Unveil](http://luis-almeida.github.io/unveil/)
*   [yall.js (另一个延迟加载器)](https://github.com/malchata/yall.js) 它只有1KB，并且同样支持浏览器“交叉观察者”API。

**延迟加载有什么问题么？**

*   一些阅读器、搜索机器人和任何禁用JavaScript的用户将无法看延迟加载的图像。但是，我们可以通过一个`<noscript>`标签来提示解决！
*   对页面滚动进行侦听，例如确定何时加载延迟加载的图像，可能会对浏览器的滚动性能产生不利影响。可能会导致浏览器重绘多次，从而减缓网络爬取的进程 - 但是，智能的延迟加载库将会使用节流阀（throttling）来缓解这种情况。一个可能的解决方案是浏览器的“交叉观察者”API，lazysizes是支持这种模式的。

延迟加载图像是减少带宽占用、降低加载消耗和改善用户体验的一个广泛使用的方式。你可以评估它，对你网站的用户体验是否有意义。需要进一步了解，可以参阅[延迟加载图像](https://jmperezperez.com/lazy-loading-images/)和[实现Medium的渐进式加载](https://jmperezperez.com/medium-image-progressive-loading-placeholder/)。


## <a id="display-none-trap" href="#display-none-trap">避免display:none的陷阱</a>

一些陈旧的响应式图像解决方案，会错误地理解当设置CSS `display`属性时浏览器处理图像请求的方式（以为`display:none`的时候就不会请求获取图像）。这可能会是你比你所期望的请求了更多的图像的另一个原因，所以`<picture>`和`<img srcset>`依然是你响应式加载图像的首选方式。

你有没有写过一种`@Media`语句，它在某些断点上将图像设置为`display:none`？

```html
<img src="img.jpg">
<style>
@media (max-width: 640px) {
    img {
        display: none;
    }
}
</style>
```

或者是使用`display:none`的CSS设置图像隐藏切换?

```html
<style>
.hidden {
  display: none;
}
</style>
<img src="img.jpg">
<img src=“img-hidden.jpg" class="hidden">
```

一个快速验证的方法，就是使用Chrome开发者工具的网络面板，可以看到上述方法想要隐藏的图像其实仍然会被获取，即使我们的期望是它们不会被访问。根据嵌入式资源规范，浏览器的这种行为其实是没有问题的。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1503260160/essential-image-optimization/display-none-images.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1503260160/essential-image-optimization/display-none-images.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503260160/essential-image-optimization/display-none-images.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503260160/essential-image-optimization/display-none-images.jpg"
        alt="Images hidden with display:none still get fetched"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1503260160/essential-image-optimization/display-none-images.jpg"/>
</noscript>
</picture>
</figure>

**使用`display:none`会避免触发图像`src`指定的链接请求?**

```html
<div style="display:none"><img src="img.jpg"></div>
```

答案是否定的。指定的图像仍然将被请求获取。样式`display:none`无法影响这个请求，因为在JavaScript可以修改src之前，图像请求就已经发出了。

**那么使用`display:none`会避免触发图像`background: url()`指定的链接请求么?**

```html
<div style="display:none">
  <div style="background: url(img.jpg)"></div>
</div>
```

这回答案是肯定的。 上面编写的元素一旦被解析，其中的CSS背景图片就不会被请求获取。CSS样式解析计算时会认为，带有`display:none`元素的任何子元素都是没有作用的，对于整个文档的显示没有影响。所以，子元素中的任何背景图片都是会被忽略并不被下载的。

Jake Archibald的[请求任务解析](https://jakearchibald.github.io/request-quest/)中对于你响应式图像加载中使用`display:none`的陷阱有一个很好的测验。当你怀疑你的浏览器是如何处理图像请求加载时，可以打开它的开发者工具并自己验证图像加载的情况。

再次提醒你，在可能的情况下，还是建议使用`<picture>`和`<img srcset>`来处理，而不是依赖于`display:none`。

## <a id="image-processing-cdns" href="#image-processing-cdns">[图像CDN服务对你有意义吗？](https://images.guide/#image-processing-cdns)</a>

*你浪费在阅读博客帖子学习如何来设置自己的图像处理流水线，并调整那些配置的时间，对于你的服务来说其实也是一种费用成本。尤其随着Cloudinary开始提供了免费服务，Imgix和作为OSS替代产品Thumbor的都提供的免费试用，你其实有大量可以使用的自动化服务可以选择。*

为了实现最佳的页面加载时间，你需要优化你的图像加载。此优化可能需要响应式图像策略，并且需要受益于服务器上的图像压缩、自动选择最佳格式和响应式调整尺寸。重要的是，您还需要尽可能快地以正确的分辨率将正确尺寸的图像提供给正确的设备。这样做可并不像人们想象的那么容易。

**使用你自己的服务器还是使用CDN服务**

由于图像处理的复杂性和可变性，我们将提供一些在该领域经验丰富的开放服务的报价，然后继续提出一些建议。

“如果你的产品不是一个专业的图像处理应用，那么你就不要自己这样做，因为像Cloudinary（或imgix，Ed。）这样的服务会比你更有效率，请更好地使用它们。如果你担心成本问题，请考虑开发和维护的花费，对比托管、存储和交付的花费。“ - [克里斯·吉米尔](https://medium.com/@cmgmyr/moving-from-self-hosted-image-service-to-cloudinary-bd7370317a0d)。


目前情况下，我们表示同意并建议您考虑使用CDN来满足您的图像处理需求。我们将审查两个CDN，将它们的功能与之前提出的图像处理任务进行比较。

**Cloudinary和imgix**

[Cloudinary](http://cloudinary.com/)和[imgix](https://www.imgix.com/)是两个已建立的图像处理CDN服务。他们是全球成千上万的开发商和公司的选择，包括了Netflix和Red Bull。让我们更详细地了解一下它们。∂

**开始之前，应该先了解那些？**

首先，除非你是像它们一样的大型服务集群的所有者，否则与反复开发你自己的解决方案相比，它们一个巨大的优势就是使用了分布式的全球网络系统，可以让你的图像副本更接近你的用户。而随着技术的进步，CDN还可以更轻松地“未来化”你的图像加载策略 - 你自己做的话可能需要很大的维护量，同时CDN还可以跟踪浏览器对新兴格式的支持并且跟进图像压缩社区的最新技术进展。

第二，每个服务都是有定价分级的，像Cloudinary提供了一个[免费的级别](http://cloudinary.com/pricing)；而imgix相对于他们的一些高价服务计划，也推出了很多成本低廉地定价。而且Imgix提供免费[试用](https://www.imgix.com/pricing)服务，这几乎与免费是相同的。

第三，API访问是提供两种服务的。开发人员可以通过编程直接调用访问CDN，并自动进行图像处理。而客户端库、框架插件和API文档也可以使用，当然其中一些功能是限于较高的费用级别的。

**现在，让我们开始吧**

这里，我们将我们的讨论限制在静态图像上。Cloud和Imgix都提供了一系列图像处理方案，并且在标准版和免费版中都支持压缩、调整尺寸、裁剪以及缩略图生成等主要功能。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1502426282/essential-image-optimization/Modern-Image36.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1502426282/essential-image-optimization/Modern-Image36.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image36.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image36.jpg"
        alt="cloudinary media library"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1502426282/essential-image-optimization/Modern-Image36.jpg"/>
</noscript>
</picture>
<figcaption>Cloudinary多媒体库：默认情况下，Cloudinary的编码是生成[非渐进式JPEG](http://cloudinary.com/blog/progressive_jpegs_and_green_martians)。要选择生成它们，请选中'更多选项'中的'进阶'选项，或者传递'fl_progressive'参数标识。</figcaption>
</figure>

Cloudinary列出了[七个应用较广的图像转换](http://cloudinary.com/documentation/image_transformations)类别，其中共有48个子类别。Imgix则声称它包含超过[100种的图像处理操作](https://docs.imgix.com/apis/url?_ga=2.52377449.1538976134.1501179780-2118608066.1501179780)。

**那默认情况下发生什么？**

Cloudinary在默认情况下会执行以下优化：

*   [使用MozJPEG编码JPEG](https://twitter.com/etportis/status/891529495336722432) (默认会使用Guetzli工具)
*   从转换后图像文件中清除所有相关的元数据（原始图像保持不变）。不想执行此行为并要生成元数据完整的转换后图像，请添加keep_iptc标识。
*   生成默认质量级别的WebP、GIF、JPEG和JPEG-XR格式。要调整默认的质量级别，请在转换时设置质量参数。
*   运行[优化](http://cloudinary.com/documentation/image_optimization#default_optimizations)算法，最小化文件体积并对PNG、JPEG或GIF格式图像的视觉质量影响最小。

Imgix并没有如Cloudinary那样的默认优化。但是，它具有可设置的默认图像质量。对于imgix的使用者来说，自动参数可帮助您在图像目录中自动执行基础的优化处理。

目前，它有[四种不同的方法](https://docs.imgix.com/apis/url/auto)：

*   压缩
*   视觉增强
*   文件格式转换
*   红眼删除

Imgix支持以下图像格式：JPEG，JPEG2000，PNG，GIF，动画GIF，TIFF，BMP，ICNS，ICO，PDF，PCT，PSD，AI。

Cloudinary支持以下图像格式：JPEG，JPEG 2000，JPEG XR，PNG，GIF，动画GIF，WebP，动画WebP，BMP，TIFF，ICO，PDF，EPS，PSD，SVG，AI，DjVu，FLIF，TARGA。

**它们的性能如何?**

CDN传输性能主要与[网络延迟](https://docs.google.com/a/chromium.org/viewer?a=v&pid=sites&srcid=Y2hyb21pdW0ub3JnfGRldnxneDoxMzcyOWI1N2I4YzI3NzE2)和网络速度相关。

对于完全未缓存的图像，延迟总会是有所增加的。但是，一旦一个图像已经被缓存并分布到网络服务器上，事实上全球的CDN可以很快找到对用户响应最快的节点，加上正确处理图像的所带来的字节节省，对比那些处理图像不佳或需要跨越地球的单个服务器来说，CDN的延迟问题大大减小。

这两个都是访问快速和使用广泛的CDN服务。配置它们可以减少延迟并提高下载速度。下载速度影响页面加载时间，而页面加载时间是提高用户体验和用户转换率的最重要指标之一。

**那么它们比较起来如何呢？**

Cloudinary拥有[160K客户](http://cloudinary.com/customers)，其中包括了Netflix、eBay和Dropbox。Imgix没有报告有多少客户，但它比Cloudinary要少一些。即使如此，imgix依然包括了一些重量级的图像用户，已知的如Kickstarter、Exposure、unsplash和Eventbrite。

实际上，在图像处理中存在如此多的不受控制的变量，对这两个服务巨头之间进行点对点的性能比较是很困难的。这很大程度上取决于处理图像需要多少时间（这是一个可变的时间量），以及影响速度和下载时间的最终输出需求的大小和分辨率。花费成本也可能最终是你最重要的因素。

CDN服务需要花费金钱。一些流量大的图像业务网站每月可能要花费数百美元的CDN费用。另外，需要一定程度的知识储备和编程技能才能充分利用这些服务。如果你不是要做什么太过分的事情，你是不会有任何麻烦的。

但是，如果您不太喜欢使用图像处理工具或API，那么你需要经历一个学习曲线。为了适应CDN服务器链接定位，你需要更改本地链接中的一些URL。你需要勤快的检查每个地址的正确性:)

**结论**

如果您正在进行或计划制作自己的图像处理服务，也许您应该考虑一下是否可以使用CDN。

## <a id="caching-image-assets" href="#caching-image-assets">[缓存图像资源](https://images.guide/#caching-image-assets)</a>

对于Web上的资源来说，可以使用[HTTP头部的Cache属性](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#cache-control)指定缓存策略。具体来说，`Cache-Control`可以定义由谁来缓存响应以及可以保存多长时间。

你向用户提供的大部分图片都是将来[不会修改的](http://kean.github.io/post/image-caching)静态资源。这种资源的最佳缓存策略就是，积极去缓存它。

在进行HTTP头的缓存设置时，设置为最长时间为一年的Cache-Control（例如`Cache-Control:public; max-age=31536000`）。这种极为积极的缓存策略对于大多数类型的图像都是很好的，尤其是像头像和图标这些长久使用的图像。

<aside class="note"><b>注意:</b>如果你是使用PHP提供图像，可能会因为使用默认的[session_cache_limiter](http://php.net/manual/en/function.session-cache-limiter.php)设置而破坏缓存规则。这可能是图像缓存的灾难，也许你会想通过设置session_cache_limiter（'public'）（即设置为public,max-age =）来[解决](https://stackoverflow.com/a/3905468)此问题。另外，禁用和设置自定义缓存控制头也可以。 </aside>

## <a id="preload-critical-image-assets" href="#preload-critical-image-assets">[预加载关键图像资源](https://images.guide/#preload-critical-image-assets)</a>

对于关键图像资源的预加载，可以使用 [`<link rel=preload>`](https://www.w3.org/TR/preload/). 

`<link rel=preload>` 是一个声明式的提取，允许你强制浏览器对资源进行请求，而不会阻止页面文档的`onload`事件。它可以增加资源请求的优先级，防止在文档解析的后期可能无法找到资源。 

可以通过指定 `as` 的值 `image`来预加载图像文件：

```html
<link rel="preload" as="image" href="logo.jpg"/>
```

`<img>`，`<picture>`，`srcset`和SVGs中使用的图像资源，都可以采用这种方式优化加载。

<aside class="note"><b>注意:</b> [支持](http://caniuse.com/#search=preload)`<link rel="preload">`浏览器包括Chrome和其他基于Blink的浏览器如Opera、[Safari的技术预览版](https://developer.apple.com/safari/technology-preview/release-notes/)，[Firefox也已经增加了支持](https://bugzilla.mozilla.org/show_bug.cgi?id=1222633)。</aside>

像[飞利浦](https://www.usa.philips.com/)、[FlipKart](https://www.flipkart.com/)和[施乐](https://www.xerox.com/)等网站都是使用`<link rel=preload>`来预加载其主要的徽标资源（通常在页面载入的早期）。[Kayak](https://kayak.com/)也使用了预加载，以确保其页面顶部的焦点图像尽快加载出来。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1504057647/essential-image-optimization/preload-philips.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1504057647/essential-image-optimization/preload-philips.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504057647/essential-image-optimization/preload-philips.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504057647/essential-image-optimization/preload-philips.jpg"
        alt="Philips use link rel=preload to preload their logo image"
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504057647/essential-image-optimization/preload-philips.jpg"/>
</noscript>
</picture>
</figure>

**HTTP头的预加载是什么？** 

可以使用HTML标签或者[页面`<header>`的Link中](https://www.w3.org/wiki/LinkHeader)指定预加载链接。在任一种情况下，预加载链接都会指示浏览器开始将资源加载到内存高速缓存中，指明该页面需要高效率的使用此资源，并且不希望等待页面加载或解析器发现它时再获取。

一个`<header>`中的预加载连接设置，就像下面这样：

```
Link: <https://example.com/logo-hires.jpg>; rel=preload; as=image
```

当“金融时报”在他们的网站网页的头部设置了一个链接预加载之后，显示他们头条图像所花费的时间被减少了[1秒钟](https://twitter.com/wheresrhys/status/843252599902167040)：

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1504055773/essential-image-optimization/preload-financial-times.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1504055773/essential-image-optimization/preload-financial-times.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504055773/essential-image-optimization/preload-financial-times.jpg" />

<img
        class="lazyload"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504055773/essential-image-optimization/preload-financial-times.jpg"
        alt="The FT using preload. Displayed are the WebPageTest before and after traces showing improvements."
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1504055773/essential-image-optimization/preload-financial-times.jpg"/>
</noscript>
</picture>
<figcaption>底部：使用了`<link rel=preload>`，顶部：没有使用。WebPageTest使用Moto G4在3G环境下对“金融时报”首页[之前](https://www.webpagetest.org/result/170319_Z2_GFR/)和[之后](https://www.webpagetest.org/result/170319_R8_G4Q/)的载入时间比较。</figcaption>
</figure>

同样地，维基百科也通过使用预加载链接技术改善了他们徽标的载入时间表现，可以通过他们的[此案例研究](https://phabricator.wikimedia.org/phame/post/view/19/improving_time-to-logo_performance_with_preload_links/)中的介绍看到。

**使用预加载时有哪些注意事项？**

首先，你要非常确定指定的图像资源是非常值得被预先加载的；如果它们对你的用户体验不是至关重要的，那么页面上可能会有其他内容需要重点关注要预先加载。通过优先处理图像请求，您可能最终会将其他资源进一步放置到队列中。

一个非常需要注意的是，要避免使用`rel=preload`预加载那些不被浏览器广泛支持的图像格式（例如WebP）。最好还要避免将其用于使用`srcset`设置的根据设备条件而变化图像链接位置的响应式图像。

想要了解更多的预加载相关的信息，请查阅一下文章：[Chrome中的预加载、预提取和优先级](https://medium.com/reloading/preload-prefetch-and-priorities-in-chrome-776165961bbf) and [预加载：究竟好在哪里？](https://www.smashingmagazine.com/2016/02/preload-what-is-it-good-for/).

## <a id="performance-budgets" href="#performance-budgets">图像的网络性能预算</a>

性能预算是站点团队尝试不超过的网页显示效果的“预先估算”。例如，“任何页面上的图像不会超过200KB”或“3秒以内的用户必须可以使用”。当一个预算没有得到满足，探索原因和如何解决以达到目标。

预算为与利益相关者讨论性能表现时提供了一个有用的框架。当一个设计或者业务决策可能会影响网站的效果时，请遵循此预算。它们是一个参考，可以在对网站的用户体验造成伤害时，提示你推迟或重新思考这一变化。

我发现，当页面性能监控自动化的时候，团队会在性能预算方面取得了最大的成功。因为这时候，团队无需手动检查网络瀑布的预算回归，而且标记预算何时交叉也会自动进行。两个比较有用的性能预算跟踪服务是[Calibre](https://calibreapp.com/docs/metrics/budgets)和[SpeedCurve](https://speedcurve.com/blog/tag/performance-budgets/)。

一旦定义了图像尺寸的性能预算，SpeedCurve将会自动开始监控它，并在超出预算时提醒您：

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1505805372/essential-image-optimization/F2BCD61B-85C5-4E82-88CF-9E39CB75C9C0.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1505805372/essential-image-optimization/F2BCD61B-85C5-4E82-88CF-9E39CB75C9C0.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1505805372/essential-image-optimization/F2BCD61B-85C5-4E82-88CF-9E39CB75C9C0.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1505805372/essential-image-optimization/F2BCD61B-85C5-4E82-88CF-9E39CB75C9C0.jpg"
        alt="SpeedCurve image size monitoring."
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1505805372/essential-image-optimization/F2BCD61B-85C5-4E82-88CF-9E39CB75C9C0.jpg"/>
</noscript>
</picture>
</figure>

Calibre也提供了类似的功能，支持为你的每个目标设备类型设置预算。这是很有用的，因为你通过WiFi在桌面上的图像尺寸的预算可能会与你在手机上的预算有很大的不同。

<figure>
<picture>
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_500/v1505805371/essential-image-optimization/budgets.jpg"
        media="(max-width: 640px)" />
<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/c_scale,w_900/v1505805371/essential-image-optimization/budgets.jpg"
        media="(max-width: 1024px)" />

<source
        data-srcset="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1505805371/essential-image-optimization/budgets.jpg" />

<img
        class="lazyload small"
        data-src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1505805371/essential-image-optimization/budgets.jpg"
        alt="Calibre supports budgets for image sizes."
         />
<noscript>
  <img src="https://res.cloudinary.com/ddxwdqwkr/image/upload/v1505805371/essential-image-optimization/budgets.jpg"/>
</noscript>
</picture>
</figure>

## <a id="closing-recommendations" href="#closing-recommendations">[最后的建议](https://images.guide/#closing-recommendations)</a>

最终，选择一个图像优化策略将取决于你为你的用户提供什么图像类型，和你决定设定一套怎样的合理的评估标准。你可能会使用SSIM或Butteraugli的评分，或者要求图像它要足够小，总之放飞你的想象去体会什么才有最有意义的。

**以下是我的最终的建议：**

如果您**无法**基于浏览器支持情况，响应符合条件的图像格式，请记得：


* 对于质量级别高于90的JPEG图像，Guetzli + MozJPEG的jpegtran是一个好格式。
    * 对于网络使用，`q=90`会有些浪费。你可以降低到`q=80`，甚至在2x显示器上可以设置为`q=50`。由于Guetzli不会将图像压缩到那么低，对于Web上图像你可以使用MozJPEG。
    * KornelLesiński最近改进了MozJPEG的cjpeg命令，用添加小的sRGB配置文件，以帮助Chrome在宽色域显示器上显示更加自然的颜色。
* 对于PNG，pngquant + advpng有非常好的速度/压缩比。

如果你**可以**有条件的响应图像格式 （请使用`<picture>`标签、[支持请求header响应](https://www.igvita.com/2013/05/01/deploying-webp-via-accept-content-negotiation/)或者使用[Picturefill](https://scottjehl.github.io/picturefill/)）：

* 为支持WebP的浏览器提供它
    * 从原始的100％质量的图像创建WebP图像。否则，您将会给支持WebP的浏览器一张带有JPEG扭曲和 WebP扭曲的看上去糟透了的图像！但是，如果你使用WebP压缩未压缩的源图像，它将几乎没有WebP扭曲，并且可以更好地压缩。
    * WebP团队使用的默认设置`-m 4 -q 75`，通常适用于大多数针对速度/比率进行优化的情况。
    * WebP还具有无损（`-m 6 -q 100`）的特殊模式，可以通过探索所有参数组合将文件减小到最小的尺寸。这是一个较慢的过程，但对于静态资源这是值得的。
* 退而求其次，可以将Guetzli或MozJPEG压缩过的原图提供给其他浏览器。

最后，祝你压缩快乐！

<aside class="note"><b>注意:</b> 关于如何优化图像的更实际的指导，我强烈推荐Jeremy Wagner的[Web性能实战](https://www.manning.com/books/web-performance-in-action)。另外，[高性能的图像](http://shop.oreilly.com/product/0636920039730.do)也有关于这个主题的优秀而细微的建议。</aside>

## <a id="trivia" href="#trivia">备注</a>

* [JPEG XT](https://jpeg.org/jpegxt/)定义了关于1992年JPEG规范的扩展。作为对于古老的JPEG上进行像素完美渲染的扩展，这个规范简化了旧的1992规范，并且选择[libjpeg-turbo](https://libjpeg-turbo.org/)作为其参考实现（基于受欢迎程度）。
* [PIK](https://github.com/google/pik)是一个值得关注的新型图像编解码器。它与JPEG兼容，并且具有更高效的颜色空间，类似于Guetzli的优势。它可以以JPEG的2/3的速度进行图像解码，并且比libjpeg提供的文件体积小54％。与Guetzli-ified JPEG相比，解码和压缩都更快。一项关于现代图像编码心理视觉相似性的[研究](https://encode.ru/threads/2814-Psychovisual-analysis-on-modern-lossy-image-codecs)表明，PIK仅为其他替代品的一半大小。不幸的是，目前来看，这个编解码器还有很长的路要走，它的编码时间现在（2017年8月）还是慢的基本无法使用。
* 通常推荐使用[ImageMagick](https://www.imagemagick.org/script/index.php)进行图像优化。这篇文章也认为它是一个很好的工具，但是它的输出通常需要更多的优化，而其他工具可以提供更好的输出。我们推荐可以尝试使用[libvps](https://github.com/jcupitt/libvips)，但它是比较低级别的工具，需要更多的技术能力才能使用。ImageMagick曾经[注意到](https://imagetragick.com/#moreinfo)您可能想知道的安全漏洞。
* Blink（Chrome使用的渲染引擎）会在主线程中解码图像。将解码工作转移到合成器线程，释放主线程以处理其他任务，我们称之为延迟解码。在延迟解码时，解码工作会保留关键路径以便在显示器上呈现出框架，因此仍可能导致动画抖动。通过API[`img.decode()`](https://html.spec.whatwg.org/multipage/embedded-content.html#dom-img-decode)应该可以帮助你解决抖动的问题。

<p class="license">本书内容基于[Attribution-NonCommercial-NoDerivs 2.0 Generic (CC BY-NC-ND 2.0)](https://creativecommons.org/licenses/by-nc-nd/2.0/)授权许可，同时代码示例是基于[Apache 2.0许可证授权](http://www.apache.org/licenses/LICENSE-2.0).。版权所有Google, 2017。</p>

</body>
</html>
