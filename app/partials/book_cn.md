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

### <a id="delivering-hidpi-with-srcset" href="#delivering-hidpi-with-srcset">Delivering HiDPI images using `srcset`</a>

Users may access your site through a range of mobile and desktop devices with high-resolution screens. The [Device Pixel Ratio](https://stackoverflow.com/a/21413366) (DPR) (also called the "CSS pixel ratio") determines how a device’s screen resolution is interpreted by CSS. DPR was created by phone manufacturers to enable increasing the resolution and sharpness of mobile screens without making elements appear too small.

To match the image quality users might expect, deliver the most appropriate resolution images to their devices. Sharp, high-DPR images (e.g. 2x, 3x) can be served to devices that support them. Low and standard-DPR images should be served to users without high-res screens as such 2x+ images will
often weigh significantly more bytes.

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
<figcaption>Device Pixel Ratio: Many sites track the DPR for popular devices including [material.io](https://material.io/devices/) and [mydevice.io](https://mydevice.io/devices/).</figcaption>
</figure>


[srcset](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images) allows a browser to select the best available image per device, e.g. selecting a 2x image for a 2x mobile display. Browsers without `srcset` support can fallback to the default `src` specified in the `<img>` tag.

```
<img srcset="paul-irish-320w.jpg,
             paul-irish-640w.jpg 2x,
             paul-irish-960w.jpg 3x"
     src="paul-irish-960w.jpg" alt="Paul Irish cameo">
```

Image CDNs like [Cloudinary](http://cloudinary.com/blog/how_to_automatically_adapt_website_images_to_retina_and_hidpi_devices) and [Imgix](https://docs.imgix.com/apis/url/dpr) both support controlling image density to serve the best
density to users from a single canonical source.

<aside class="note"><b>Note:</b> You can learn more about Device Pixel Ratio and responsive images in this free [Udacity](https://www.udacity.com/course/responsive-images--ud882) course and the [Images](https://developers.google.com/web/fundamentals/design-and-ui/responsive/images) guide on Web Fundamentals.</aside>

A friendly reminder that [Client Hints](https://www.smashingmagazine.com/2016/01/leaner-responsive-images-client-hints/) can also provide an alternative to specifying each possible pixel density and format in your responsive image markup. Instead, they append this information to the HTTP request so web servers can pick the best fit for the current device's screen density.

### <a id="art-direction" href="#art-direction">Art direction</a>

Although shipping the right resolution to users is important, some sites also need to think about this in terms of **[art direction](http://usecases.responsiveimages.org/#art-direction)**. If a user is on a smaller screen, you may want to crop or zoom in and display the subject to make best use of available space. Although art direction is outside the scope of this write-up, services like[ Cloudinary](http://cloudinary.com/blog/automatically_art_directed_responsive_images%20) provide APIs to try automating this as much as possible.

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
<figcaption>Art direction: Eric Portis put together an excellent [sample](https://ericportis.com/etc/cloudinary/) of how responsive images can be used for art-direction. This example adapts the main hero image's visual characteristics at different breakpoints to make best use of the available space.</figcaption>
</figure>

## <a id="color-management" href="#color-management">Color management</a>

There are at least three different perspectives of color: biology, physics and print. In biology, color is a [perceptual phenomenon](http://hubel.med.harvard.edu/book/ch8.pdf). Objects reflect light in different combinations of wavelengths. Light receptors in our eyes translate these wavelengths into the sensation we know as color. In physics, it’s light that matters - light frequencies and brightness. Print is more about color wheels, inks and artistic models.

Ideally, every screen and web browser in the world would display color exactly the same. Unfortunately, due to a number of inherent inconsistencies, they don’t. Color management allows us to reach a compromise on displaying color through color models, spaces and profiles.

#### Color models

[Color models](https://en.wikipedia.org/wiki/Gamma_correction) are a system for generating a complete range of colors from a smaller set of primary colors. There are different types of color spaces which use different parameters to control colors. Some color spaces have fewer control parameters than others - e.g. grayscale only has a single parameter for controlling brightness between black and white colors. 

Two common color models are additive and subtractive. Additive color models (like RGB, used for digital displays) use light to show color while subtractive color models (like CMYK, used in printing) work by taking light away.

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
<figcaption>In RGB red, green and blue light are added in different combinations to produce a broad spectrum of colors. CYMK (cyan, magenta, yellow and black) works through different colors of ink subtracting brightness from white paper.  </figcaption>
</figure>

[Understanding Color Models and Spot Color Systems](https://www.designersinsights.com/designer-resources/understanding-color-models/) has a good description of other color models and modes, such as HSL, HSV and LAB.

#### Color spaces

[Color spaces](http://www.dpbestflow.org/color/color-space-and-color-profiles#space) are a specific range of colors that can be represented for a given image. For example, if an image contains up to 16.7 million colors, different color spaces allow the use of narrower or wider ranges of these colors. Some developers refer to color models and color spaces as the same thing.

[sRGB](https://en.wikipedia.org/wiki/SRGB) was designed to be a [standard](https://www.w3.org/Graphics/Color/sRGB.html) color space for the web and is based on RGB. It’s a small color space that is typically considered the lowest common denominator and is the safest option for color management cross-browser. Other color spaces (such as [Adobe RGB](https://en.wikipedia.org/wiki/Adobe_RGB_color_space) or [ProPhoto RGB](https://en.wikipedia.org/wiki/ProPhoto_RGB_color_space) - used in Photoshop and Lightroom) can represent more vibrant colors than sRGB but as the latter is more ubiquitous across most web browsers, games and monitors, it’s what is generally focused on.

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
<figcaption>Above we can see a visualization of gamut - the range of colors a color space can define.</figcaption>
</figure>

Color spaces have three channels (red, green and blue). There are 255 colors possible in each channel under 8-bit mode, bringing us to a total of 16.7 million colors. 16-bit images can show trillions of colors. 

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
<figcaption>A comparison of sRGB, Adobe RGB and ProPhoto RGB using an image from [Yardstick](https://yardstick.pictures/tags/img%3Adci-p3). It's incredibly hard to show this concept in sRGB, when you can't show colors that can't be seen. A regular photo in sRGB vs wide gamut should have everything identical, except most saturated "juicy" colors.</figcaption>
</figure>

The differences in color spaces (like sRGB, Adobe RGB and ProPhoto RGB) are their gamut (the range of colors they can reproduce with shades), illuminant and [gamma](http://blog.johnnovak.net/2016/09/21/what-every-coder-should-know-about-gamma/) curves. sRGB is ~20% smaller than Adobe RGB and ProPhoto RGB is ~[50% larger](http://www.petrvodnakphotography.com/Articles/ColorSpace.htm) than Adobe RGB. The above image sources are from [Clipping Path](http://clippingpathzone.com/blog/essential-photoshop-color-settings-for-photographers).

[Wide-gamut](http://www.astramael.com/) is a term describing color spaces with a gamut larger than sRGB. These types of displays are becoming more common. That said, many digital displays are still simply unable to display color profiles that are significantly better than sRGB. When saving for the web in Photoshop, consider using the 'Convert to sRGB’ option unless targeting users with higher-end wide-gamut screens. 

<aside class="key-point"><b>Note:</b> When working with original photography, avoid using sRGB as your primary color space. It's smaller than the color spaces most cameras support and can cause clipping. Instead, work on a larger color space (like ProPhoto RGB) and output to sRGB when exporting for the web.</aside>

**Are there any cases where wide gamut makes sense for web content?**

Yes. If an image contains very saturated/juicy/vibrant color and you care about it being just as juicy on screens that support it. However, in real photos that rarely happens. Often it's easy to tweak color to make it appear vibrant, without it actually exceeding sRGB gamut 

That's because human color perception is not absolute, but relative to our surroundings and is easily fooled. If your image contains a fluorescent highlighter color, then you'll have an easier time with wide gamut.

#### Gamma correction and compression

[Gamma correction](https://en.wikipedia.org/wiki/Gamma_correction) (or just Gamma) controls the overall brightness of an image. Changing the gamma can also alter the ratio of red to green and blue colors. Images without gamma correction can look like their colors are bleached out or too dark. 

In video and computer graphics, gamma is used for compression, similar to data compression. This allows you to squeeze useful levels of brightness in fewer bits (8-bit rather than 12 or 16). Human perception of brightness is not linearly proportional to physical amount of light. Representing colors in their true physical form would be wasteful when encoding images for human eyes. Gamma compression is used to encode brightness on a scale that is closer to human perception. 

With gamma compression useful scale of brightness fits in 8 bits of precision (0-255 used by most RGB colors). All of this comes from the fact that if colors used some unit with 1:1 relationship to physics, RGB values would be from 1 to million where values 0-1000 would look distinct, but values between 999000-1000000 would look identical. Imagine being in a dark room where there is just 1 candle. Light a second candle and you notice significant increases in brightness in the room light. Add a third candle and it’ll seem even brighter. Now imagine being in a room with 100 candles. Light the 101st candle, the 102nd. You won’t notice a change in brightness. 

Even though in both cases, physically, exactly the same amount of light was added. So because eyes are less sensitive when light is bright, gamma compression "compresses" bright values, so in physical terms bright levels are less precise but the scale is adjusted for humans so from the human perspective all values are equally precise. 

<aside class="key-point"><b>Note:</b> Gamma compression/correction here is different to the image gamma curves you might configure in Photoshop. When gamma compression works as it should, it doesn't look like anything.</aside>

#### Color profiles

A color profile is the information describing what that the color space of a device is. It’s used to convert between different color spaces. Profiles attempt to ensure an image looks as similar as possible on these different kinds of screens and mediums. 

Images can have an embedded color profile as described by the [International Color Consortium](http://www.color.org/icc_specs2.xalter) (ICC) to represent precisely how colors should appear. This is supported by different formats including JPEGs, PNGs, SVGs and [WebP](https://developers.google.com/speed/webp/docs/riff_container) and most major browsers support embedded ICC profiles. When an image is displayed in an app and it knows the monitor's capabilities, these colors can be adjusted based on the color profile. 

<aside class="key-point"><b>Note:</b> Some monitors have a color profile similar to sRGB and cannot display much better profiles so depending on your target users displays, there may be limited value in embedding them. Check who your target users are.</aside>

Embedded color profiles can also heavily increase the size of your images (100KB+ occasionally) so be careful with embedding. Tools like ImageOptim will actually [automatically](https://imageoptim.com/color-profiles.html) remove color profiles if it finds them. In contrast, with the ICC profile removed in the name of size reduction, browsers will be forced to display the image in your monitor's color space which can lead to differences in expected saturation and contrast. Evaluate the trade-offs here make sense for your use case.

[Nine Degrees Below](https://ninedegreesbelow.com/photography/articles.html) have an excellent set of resources on ICC profile color management if you are interested in learning more about profiles.

#### Color profiles and web browsers

Earlier versions of Chrome did not have great support for color management, but this is improving in 2017 with [Color Correct Rendering](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/ptuKdRQwPAo). Displays that are not sRGB (newer MacBook Pros) will convert colors from sRGB to the display profile. This will mean colors should look more similar across different systems and browsers. Safari, Edge and Firefox can now also take ICC profiles into account, so images with a different color profile (e.g. ICC) can now display them correctly whether your screen has wide gamut or not.

<aside class="key-point"><b>Note:</b> For a great guide on how color applies to a broader spectrum of ways we work on the web, see the [nerd’s guide to color on the web](https://css-tricks.com/nerds-guide-color-web/) by Sarah Drasner.</aside>

## <a id="image-sprites" href="#image-sprites">Image spriting</a>

[Image sprites](https://developers.google.com/web/fundamentals/design-and-ui/responsive/images#use_image_sprites) (or CSS sprites) have a long history on the web, are supported by all browsers and have been a popular way to reduce the number of images a page loads by combining them into a single larger image that is sliced.

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
<figcaption>Image sprites are still widely used in large, production sites, including the Google homepage.</figcaption>
</figure>

Under HTTP/1.x, some developers used spriting to reduce HTTP requests. This came with a number of benefits, however care was needed as you quickly ran into challenges with cache-invalidation - changes to any small part of an image sprite would invalidate the entire image in a user's cache.

Spriting may now however be an [HTTP/2](https://hpbn.co/http2/) anti-pattern. With HTTP/2, it may be best to [load individual images](https://deliciousbrains.com/performance-best-practices-http2/) since multiple requests within a single connection are now possible. Measure to evaluate whether this is the case for your own network setup.

## <a id="lazy-load-non-critical-images" href="#lazy-load-non-critical-images">Lazy-load non-critical images</a>

Lazy loading is a web performance pattern that delays the loading of images in the browser until the user needs to see it. One example is, as you scroll, images load asynchronously on demand. This can further compliment the byte-savings you see from having an image compression strategy.


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

Images that must appear "above the fold," or when the web page first appears are loaded straight away. The images which follow "below the fold," however, are not yet visible to the user. They do not have to be immediately loaded into the browser. They can be loaded later — or lazy loaded — only if and when the user scrolls down and it becomes necessary to show them.

Lazy loading is not yet natively supported in the browser itself (although there have been [discussions](https://discourse.wicg.io/t/a-standard-way-to-lazy-load-images/1153/10) about it in the past). Instead, we use JavaScript to add this capability.

**Why is Lazy Loading Useful?**

This "lazy" way of loading images only if and when necessary has many benefits:

* **Reduced data consumption**: As you aren’t assuming the user will need every image fetched ahead of time, you’re only loading the minimal number of resources. This is always a good thing, especially on mobile with more restrictive data plans.
* **Reduced battery consumption**: Less workload for the user’s browser which can save on battery life.
* **Improved download speed**: Decreasing your overall page load time on an image heavy website from several seconds to almost nothing is a tremendous boost to user experience. In fact, it could be the difference between a user staying around to enjoy your site and just another bounce statistic.

**But like all tools, with great power comes great responsibility.**

**Avoid lazy-loading images above the fold.** Use it for long-lists of images (e.g. products) or lists of user avatars. Don’t use it for the main page hero image. Lazy-loading images above the fold can make loading visibly slower, both technically and for human perception. It can kill the browser’s preloader, progressive loading and the JavaScript can create extra work for the browser.

**Be very careful lazy-loading images when scrolling.**  If you wait until the user is scrolling they are likely to see placeholders and may eventually get images, if they haven’t already scrolled past them. One recommendation would be to start lazy-loading after the above-the-fold images have loaded, loading all of the images independent of user interaction.

**Who Uses Lazy Loading?**

For examples of lazy loading, look at most any major site that hosts a lot of images. Some notable sites are [Medium](https://medium.com/) and [Pinterest](https://www.pinterest.com/).

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
<figcaption>An example of Gaussian-blurred inline previews for images on Medium.com</figcaption>
</figure>

A number of sites (such as Medium) display a small, Gaussian-blurred inline preview (a few 100 bytes) that transitions (lazy-loads) to a full-quality image once it has been fetched.

José M. Pérez has written about how to implement the Medium effect using [CSS filters](https://jmperezperez.com/medium-image-progressive-loading-placeholder/) and experimented with [different image formats](https://jmperezperez.com/webp-placeholder-images/) to support such placeholders. Facebook also did a write-up on their famous 200-byte approach for such placeholders for their [cover photos](https://code.facebook.com/posts/991252547593574/the-technology-behind-preview-photos/) that is worth a read. If you’re a Webpack user, [LQIP loader](https://lqip-loader.firebaseapp.com/) can help automate some of this work away.

In fact, you can search for your favorite source of high-res photos and then scroll down the page. In almost all cases you'll experience how the website loads only a few full-resolution images at a time, with the rest being placeholder colors or images. As you continue to scroll, the placeholder images are replaced with full-resolution images. This is lazy loading in action.

**How Can I Apply Lazy Loading to My Pages?**

There are a number of techniques and plugins available for lazy loading. I recommend [lazysizes](https://github.com/aFarkas/lazysizes) by Alexander Farkas because of its decent performance, features, its optional integration with [Intersection Observer](https://developers.google.com/web/updates/2016/04/intersectionobserver), and support for plugins.

**What Can I Do with Lazysizes?**

Lazysizes is a JavaScript library. It requires no configuration. Download the minified js file and include it in your webpage.


Here is some example code taken from the README file:

Add the class "lazyload" to your images/iframes in conjunction with a data-src and/or data-srcset attribute.

Optionally you can also add a src attribute with a low quality image:

```html
<!-- non-responsive: -->
<img data-src="image.jpg" class="lazyload" />

<!-- responsive example with automatic sizes calculation: -->
<img
    data-sizes="auto"
    data-src="image2.jpg"
    data-srcset="image1.jpg 300w,
    image2.jpg 600w,
    image3.jpg 900w" class="lazyload" />

<!-- iframe example -->

<iframe frameborder="0"
    class="lazyload"
    allowfullscreen=""
    data-src="//www.youtube.com/embed/ZfV-aYdU4uE">
</iframe>
```

For the web version of this book, I paired Lazysizes (although you can use any alternative)
with Cloudinary for on-demand responsive images. This allowed me the freedom to experiment
with different values for scale, quality, format and whether or not to progressively load
with minimal effort:

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

**Lazysizes features include:**

* Automatically detects visibility changes on current and future lazyload elements
* Includes standard responsive image support (picture and srcset)
* Adds automatic sizes calculation and alias names for media queries feature
* Can be used with hundreds of images/iframes on CSS and JS-heavy pages or web apps
* Extendable: Supports plugins
* Lightweight but mature solution
* SEO improved: Does not hide images/assets from crawlers

**More Lazy Loading Options**

Lazysizes is not your only option. Here are more lazy loading libraries:

*   [Lazy Load XT](http://ressio.github.io/lazy-load-xt/)
*   [BLazy.js](https://github.com/dinbror/blazy) (or [Be]Lazy)
*   [Unveil](http://luis-almeida.github.io/unveil/)
*   [yall.js (Yet Another Lazy Loader)](https://github.com/malchata/yall.js) which is ~1KB and uses Intersection Observer where supported.

**What's the catch with Lazy Loading?**

*   Screen readers, some search bots and any users with JavaScript disabled will not be able to view images lazy loaded with JavaScript. This is however something that we can work around with a `<noscript>` fallback.
*   Scroll listeners, such as used for determining when to load a lazy-loaded image, can have an adverse impact on browser scrolling performance. They can cause the browser to redraw many times, slowing the process to a crawl - however, smart lazy loading libraries will use throttling to mitigate this. One possible solution is Intersection Observer, which is supported by lazysizes.

Lazy loading images is a widespread pattern for reducing bandwidth, decreasing costs, and improving user experience. Evaluate whether it makes sense for your experience. For further
reading see [lazy loading images](https://jmperezperez.com/lazy-loading-images/) and [implementing Medium's progressive loading](https://jmperezperez.com/medium-image-progressive-loading-placeholder/).


## <a id="display-none-trap" href="#display-none-trap">Avoiding the display:none trap</a>

Older responsive image solutions have mistaken how browsers handle image requests when setting the CSS  `display` property. This can cause significantly more images to be requested than you might be expecting and is another reason `<picture>` and `<img srcset>` are preferred for loading responsive images.

Have you ever written a media query that sets an image to `display:none` at certain breakpoints?

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

Or toggled what images are hidden using a `display:none` class?

```html
<style>
.hidden {
  display: none;
}
</style>
<img src="img.jpg">
<img src=“img-hidden.jpg" class="hidden">
```

A quick check against the Chrome DevTools network panel will verify that images hidden using these approaches still get fetched, even when we expect them not to be. This behavior is actually correct per the embedded resources spec.

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

**Does `display:none` avoid triggering a request for an image `src`?**

```html
<div style="display:none"><img src="img.jpg"></div>
```

No. The image specified will still get requested. A library cannot rely on display:none here as the image will be requested before JavaScript can alter the src.

**Does `display:none` avoid triggering a request for a `background: url()`?**

```html
<div style="display:none">
  <div style="background: url(img.jpg)"></div>
</div>
```

Yes. CSS backgrounds aren’t fetched as soon as an element is parsed. Calculating CSS styles for children of elements with `display:none` would be less useful as they don’t impact rendering of the document. Background images on child elements are not calculated nor downloaded.

Jake Archibald’s [Request Quest](https://jakearchibald.github.io/request-quest/) has an excellent quiz on the pitfalls of using `display:none` for your responsive images loading. When in doubt about how specific browser’s handle image request loading, pop open their DevTools and verify for yourself.

Again, where possible, use `<picture>` and `<img srcset>` instead of relying on `display:none`.

## <a id="image-processing-cdns" href="#image-processing-cdns">Does an image processing CDN make sense for you?</a>

*The time you'll spend reading the blog posts to setup your own image processing pipeline and tweaking your config is often >> the fee for a service. With [Cloudinary](http://cloudinary.com/) offering a free service, [Imgix](https://www.imgix.com/) a free trial and [Thumbor](https://github.com/thumbor/thumbor) existing as an OSS alternative, there are plenty of options available to you for automation.*

To achieve optimal page load times, you need to optimize your image loading. This optimization calls for a responsive image strategy and can benefit from on-server image compression, auto-picking the best format and responsive resizing. What matters is that you deliver the correctly sized image to the proper device in the proper resolution as fast as possible. Doing this is not as easy as one might think.

**Using Your Server vs. a CDN**

Because of the complexity and ever-evolving nature of image manipulation, we're going to offer a quote from someone with experience in the field, then proceed with a suggestion.

"If your product is not image manipulation, then don't do this yourself. Services like Cloudinary [or imgix, Ed.] do this much more efficiently and much better than you will, so use them. And if you're worried about the cost, think about how much it'll cost you in development and upkeep, as well as hosting, storage, and delivery costs." — [Chris Gmyr](https://medium.com/@cmgmyr/moving-from-self-hosted-image-service-to-cloudinary-bd7370317a0d)


For the moment, we are going to agree and suggest that you consider using a CDN for your image processing needs. Two CDNs will be examined to see how they compare relative to the list of tasks we raised earlier.

**Cloudinary and imgix**

[Cloudinary](http://cloudinary.com/) and [imgix](https://www.imgix.com/) are two established image processing CDNs. They are the choice of hundreds of thousands of developers and companies worldwide, including Netflix and Red Bull. Let's look at them in more detail.

**What are the Basics?**

Unless you are the owner of a network of servers like they are, their first huge advantage over rolling your own solution is that they use a distributed global network system to bring a copy of your images closer to your users. It's also far easier for a CDN to "future proof" your image loading strategy as trends change - doing this on your own requires maintenance, tracking browser support for emerging formats & following the image compression community.

Second, each service has a tiered pricing plan, with Cloudinary offering a [free level](http://cloudinary.com/pricing) and imgix pricing their standard level inexpensively, relative to their high-volume premium plan. Imgix offers a free [trial](https://www.imgix.com/pricing) with a credit towards services, so it almost amounts to the same thing as a free level.

Third, API access is provided by both services. Developers can access the CDN programmatically and automate their processing. Client libraries, framework plugins, and API documentation are also available, with some features restricted to higher paid levels.

**Let's Get to the Image Processing**

For now, let's limit our discussion to static images. Both Cloudinary and Imgix offer a range of image manipulation methods, and both support primary functions such as compression, resizing, cropping and thumbnail creation in their standard and free plans.

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
<figcaption>Cloudinary Media Library: By default Cloudinary encodes [non-Progressive JPEGs](http://cloudinary.com/blog/progressive_jpegs_and_green_martians). To opt-in to generating them, check the 'Progressive' option in 'More options' or pass the 'fl_progressive' flag.</figcaption>
</figure>

Cloudinary lists [seven broad image transformation](http://cloudinary.com/documentation/image_transformations) categories, with a total of 48 subcategories within them. Imgix advertises over [100 image processing operations](https://docs.imgix.com/apis/url?_ga=2.52377449.1538976134.1501179780-2118608066.1501179780).

**What Happens by Default?**

*   Cloudinary performs the following optimizations by default:
*   [Encodes JPEGs using MozJPEG](https://twitter.com/etportis/status/891529495336722432) (opted against Guetzli as a default)
*   Strips all associated metadata from the transformed image file (the original image is left untouched). To override this behavior and deliver a transformed image with its metadata intact, add the keep_iptc flag.
*   Can generate WebP, GIF, JPEG, and JPEG-XR formats with automatic quality. To override the default adjustments, set the quality parameter in your transformation.
*   Runs [optimization](http://cloudinary.com/documentation/image_optimization#default_optimizations) algorithms to minimize the file size with minimal impact to visual quality when generating images in the PNG, JPEG or GIF format.

Imgix has no default optimizations such as Cloudinary has. It does have a settable default image quality. For imgix, auto parameters help you automate your baseline optimization level across your image catalog.

Currently, it has [four different methods](https://docs.imgix.com/apis/url/auto):

*   Compression
*   Visual enhancement
*   File format conversion
*   Redeye removal

Imgix supports the following image formats: JPEG, JPEG2000, PNG, GIF, Animated GIF, TIFF, BMP, ICNS, ICO, PDF, PCT, PSD, AI

Cloudinary supports the following image formats: JPEG, JPEG 2000, JPEG XR, PNG, GIF, Animated GIF, WebP, Animated WebP,BMPs, TIFF, ICOs, PDF, EPS, PSD, SVG, AI, DjVu, FLIF, TARGA.

**What About Performance?**

CDN delivery performance is mostly about [latency](https://docs.google.com/a/chromium.org/viewer?a=v&pid=sites&srcid=Y2hyb21pdW0ub3JnfGRldnxneDoxMzcyOWI1N2I4YzI3NzE2) and speed.

Latency always increases somewhat for completely uncached images. But once an image is cached and distributed among the network servers, the fact that a global CDN can find the shortest hop to the user, added to the byte savings of a properly-processed image, almost always mitigates latency issues when compared to poorly processed images or solitary servers trying to reach across the planet.

Both services use fast and wide CDN. This configuration reduces latency and increases download speed. Download speed affects page load time, and this is one of the most important metrics for both user experience and conversion.

**So How Do They Compare?**

Cloudinary has [160K customers](http://cloudinary.com/customers) including Netflix, eBay and Dropbox. Imgix doesn't report how many customers it has, but it is smaller than Cloudinary. Even so, imgix's base includes heavyweight image users such as Kickstarter, Exposure, unsplash, and Eventbrite.  

There are so many uncontrolled variables in image manipulation that a head-to-head performance comparison between the two services is difficult. So much depends on how much you need to process the image — which takes a variable amount of time — and what size and resolution are required for the final output, which affects speed and download time. Cost may ultimately be the most important factor for you.

CDNs cost money. An image heavy site with a lot of traffic could cost hundreds of US dollars a month in CDN fees. There is a certain level of prerequisite knowledge and programming skill required to get the most out of these services. If you are not doing anything too fancy, you're probably not going to have any trouble.

But if you're not comfortable working with image processing tools or APIs, then you are looking at a bit of a learning curve. In order to accommodate the CDN server locations, you will need to change some URLs in your local links. Do the right due diligence :)

**Conclusion**

If you are currently serving your own images or planning to, perhaps you should give a CDN some consideration.

## <a id="caching-image-assets" href="#caching-image-assets">Caching image assets</a>

Resources can specify a caching policy using [HTTP cache headers](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching#cache-control). Specifically, `Cache-Control` can define who can cache responses and for how long

Most of the images you deliver to users are static assets that will[ not change](http://kean.github.io/post/image-caching) in the future. The best caching strategy for such assets is aggressive caching.

When setting your HTTP caching headers, set Cache-Control with a max-age of a year (e.g. `Cache-Control:public; max-age=31536000`). This type of aggressive caching works well for most types of images, especially those that are long-lived like avatars and image headers.

<aside class="note"><b>Note:</b> If you're serving images using PHP, it can destroy caching due to the default [session_cache_limiter](http://php.net/manual/en/function.session-cache-limiter.php) setting. This can be a disaster for image caching and you may want to [work around](https://stackoverflow.com/a/3905468) this by setting session_cache_limiter('public') which will set public, max-age=. Disabling and setting custom cache-control headers is also fine.</aside>

## <a id="preload-critical-image-assets" href="#preload-critical-image-assets">Preloading critical image assets</a>

Critical image assets can be preloaded using [`<link rel=preload>`](https://www.w3.org/TR/preload/). 

`<link rel=preload>` is a declarative fetch, allowing you to force the browser to make a request for a resource without blocking the document’s `onload` event. It enables increasing the priority of requests for resources that might otherwise not be discovered until later in the document parsing process. 

Images can be preloaded by specifying an `as` value of `image`:

```html
<link rel="preload" as="image" href="logo.jpg"/>
```

Image resources for `<img>`, `<picture>`, `srcset` and SVGs can all take advantage of this optimization.

<aside class="note"><b>Note:</b> `<link rel="preload">` is [supported](http://caniuse.com/#search=preload) in Chrome and Blink-based browsers like Opera, [Safari Tech Preview](https://developer.apple.com/safari/technology-preview/release-notes/) and has been [implemented](https://bugzilla.mozilla.org/show_bug.cgi?id=1222633) in Firefox.</aside>

Sites like [Philips](https://www.usa.philips.com/), [FlipKart](https://www.flipkart.com/) and [Xerox](https://www.xerox.com/) use `<link rel=preload>` to preload their main logo assets (often used early in the document). [Kayak](https://kayak.com/) also uses preload to ensure the hero image for their header is loaded as soon as possible.

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

**What is the Link preload header?** 

A preload link can be specified using either an HTML tag or an [HTTP Link header](https://www.w3.org/wiki/LinkHeader). In either case, a preload link directs the browser to begin loading a resource into the memory cache, indicating that the page expects with high confidence to use the resource and doesn’t want to wait for the preload scanner or the parser to discover it.

A Link preload header for images would look similar to this:

```
Link: <https://example.com/logo-hires.jpg>; rel=preload; as=image
```

When the Financial Times introduced a Link preload header to their site, they shaved [1 second off](https://twitter.com/wheresrhys/status/843252599902167040) the time it took to display their masthead image:

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
<figcaption>Bottom: with `<link rel=preload>`, Top: without. Comparison for a Moto G4 over 3G on WebPageTest both [before](https://www.webpagetest.org/result/170319_Z2_GFR/) and [after](https://www.webpagetest.org/result/170319_R8_G4Q/).</figcaption>
</figure>

Similarly, Wikipedia improved time-to-logo performance with the Link preload header as covered in their [case study](https://phabricator.wikimedia.org/phame/post/view/19/improving_time-to-logo_performance_with_preload_links/).

**What caveats should be considered when using this optimization?**

Be very certain that it's worth preloading image assets as, if they aren't critical to your user experience, there may be other content on the page worth focusing your efforts on loading earlier instead. By prioritizing image requests, you may end up pushing other resources further down the queue.

It's important to avoid using `rel=preload` to preload image formats without broad browser support (e.g. WebP). It's also good to avoid using it for responsive images defined in `srcset` where the retrieved source may vary based on device conditions. 

To learn more about preloading, see [Preload, Prefetch and Priorities in Chrome](https://medium.com/reloading/preload-prefetch-and-priorities-in-chrome-776165961bbf) and [Preload: What Is It Good For?](https://www.smashingmagazine.com/2016/02/preload-what-is-it-good-for/).

## <a id="performance-budgets" href="#performance-budgets">Web Performance Budgets For Images</a>

A performance budget is a "budget" for web page performance that a team attempts to not exceed. For example, "images will not exceed 200KB on any page" or "the user experience must be usable in under 3 seconds". When a budget isn't being met, explore why this is and how you get back on target.

Budgets provide a useful framework for discussing performance with stakeholders. When a design or business decision may impact site performance, consult the budget. They're a reference for pushing back or rethinking the change when it can harm a site's user experience.

I've found teams have the best success with performance budgets when monitoring them is automated. Rather than manually inspecting network waterfalls for budget regressions, automation can flag when the budget is crossed. Two such services that are useful for performance budget tracking are [Calibre](https://calibreapp.com/docs/metrics/budgets) and [SpeedCurve](https://speedcurve.com/blog/tag/performance-budgets/).

Once a performance budget for image sizes is defined, SpeedCurve starts monitoring and alerts you if the budget is exceeded:

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

Calibre offers a similar feature with support for setting budgets for each device-class you’re targeting. This is useful as your budget for image sizes on desktop over WiFi may vary heavily to your budgets on mobile.

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

## <a id="closing-recommendations" href="#closing-recommendations">Closing recommendations</a>

Ultimately, choosing an image optimization strategy will come down to the types of images you're serving down to your users and what you decide is a reasonable set of evaluation criteria. It might be using SSIM or Butteraugli or, if it's a small enough set of images, going off of human perception for what makes the most sense.

**Here are my closing recommendations:**

If you **can't** invest in conditionally serving formats based on browser support:


* Guetzli + MozJPEG's jpegtran is a good format for JPEG quality > 90.
    * For the web `q=90` is wastefully high. You can get away with `q=80`, and on 2x displays even with `q=50`. Since Guetzli doesn't go that low, for the web you can MozJPEG.
    * Kornel Lesi&#x144;ski recently improved mozjpeg's cjpeg command to add tiny sRGB profile to help Chrome display natural color on wide-gamut displays
* PNG pngquant + advpng has a pretty good speed/compression ratio
* If you **can** conditionally serve (using `<picture>`, the [Accept header](https://www.igvita.com/2013/05/01/deploying-webp-via-accept-content-negotiation/) or [Picturefill](https://scottjehl.github.io/picturefill/)):
    * Serve WebP down to browsers that support it
        * Create WebP images from original 100% quality images. Otherwise you'll be giving browsers that do support it worse-looking images with JPEG distortions *and* WebP distortions! If you compress uncompressed source images using WebP it'll have the less visible WebP distortions and can compress better too.
        * The default settings the WebP team use of `-m 4 -q 75` are usually good for most cases where they optimize for speed/ratio.
        * WebP also has a special mode for lossless (`-m 6 -q 100`) which can reduce a file to its smallest size by exploring all parameter combinations. It's an order of magnitude slower but is worth it for static assets.
    * As a fallback, serve Guetzli/MozJPEG compressed sources to other browsers

Happy compressing!

<aside class="note"><b>Note:</b> For more practical guidance on how to optimize images, I heavily recommend [Web Performance in Action](https://www.manning.com/books/web-performance-in-action) by Jeremy Wagner. [High Performance Images](http://shop.oreilly.com/product/0636920039730.do) is also filled with excellent, nuanced advice on this topic.</aside>

## <a id="trivia" href="#trivia">Trivia</a>

* [JPEG XT](https://jpeg.org/jpegxt/) defines extensions to the 1992 JPEG specification. For extensions to have pixel-perfect rendering on-top of old JPEG, the specification had to clarify the old 1992 spec and [libjpeg-turbo](https://libjpeg-turbo.org/) was chosen as its reference implementation (based on popularity). 
* [PIK](https://github.com/google/pik) is a new image codec worth keeping an eye on. It's compatible with JPEG, has a more efficient color-space and utilizes similar benefits found in Guetzli. It decodes at 2/3 the speed of JPEG and offers 54% more file savings than libjpeg does. It is both faster to decode and compress than Guetzli-ified JPEGs. A [study](https://encode.ru/threads/2814-Psychovisual-analysis-on-modern-lossy-image-codecs) on psychovisual similarity of modern image codes showed PIK was less than half the size of alternatives. Unfortunately, it's still early days for the codec and encoding is unusably slow at this time (August, 2017).
* [ImageMagick](https://www.imagemagick.org/script/index.php) is often recommended for image optimization. This write-up considers it a fine tool, but its output generally requires more optimization and other tools can offer better output. We recommend trying [libvps](https://github.com/jcupitt/libvips) instead, however it is lower-level and requires more technical skill to use. ImageMagick has also historically had [noted](https://imagetragick.com/#moreinfo) security vulnerabilities you may want to be aware of.
* Blink (the rendering engine used by Chrome) decodes images off the main thread. Moving the decode work to the compositor thread frees-up the main thread to work on other tasks. We call this deferred decoding. With deferred decoding, the decode work remains on the critical path for presenting a frame to the display, so it can still cause animation jank. The [`img.decode()`](https://html.spec.whatwg.org/multipage/embedded-content.html#dom-img-decode) API should help with the jank problem.

<p class="license">The content of this book is licensed under the  Creative Commons [Attribution-NonCommercial-NoDerivs 2.0 Generic (CC BY-NC-ND 2.0)](https://creativecommons.org/licenses/by-nc-nd/2.0/) license, and code samples are licensed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0). Copyright Google, 2017.</p>

</body>
</html>
