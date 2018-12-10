# 图片优化
对于一个站点来说，图片往往占据了大部分流量。对于使用CDN按流量计费的站点来说，如果能做好图片优化，可以节省一大笔费用。

本文所讨论的优化方法仅针对移动端。

# webp
首先从图片格式方面着手，webp（[google官方网址](https://developers.google.com/speed/webp/)）是谷歌推出的一种图片格式，优点在于同等画面质量下，体积比jpg、png少了25%以上。

虽然webp格式的图片相对于png和jpg体积小质量高，但是目前的兼容性在全球范围只达到了72%左右。（[caniuse](https://caniuse.com/#search=webp)截止20181210）

# webp兼容方案
### 方法一
`var isSupportWebp = document.createElement('canvas').toDataURL('image/webp').indexOf('data:image/webp') === 0;`

### 方法二
原理也是一样的，就是加载一张webp图片，如果可以加载出来，那么就是支持webp，否则就是不支持。如果支持webp，则在cookie中设置标志，并且在html根元素上添加一个class(webps)。

    ;(function(doc) {
		// 给html根节点加上webps类名
		function addRootTag() {
			doc.documentElement.className += " webps";
		}

		// 判断是否有webps=A这个cookie
		if (!/(^|;\s?)webps=A/.test(document.cookie)) {
			var image = new Image();

			// 图片加载完成时候的操作
			image.onload = function() {

				// 图片加载成功且宽度为1，那么就代表支持webp了，因为这张base64图是webp格式。如果不支持会触发image.error方法
				if (image.width == 1) {

					// html根节点添加class，并且埋入cookie
					addRootTag();
					document.cookie = "webps=A; max-age=31536000; domain=haorooms.com";
				}
			};

			// 一张支持alpha透明度的webp的图片，使用base64编码
			image.src = 'data:image/webp;base64,UklGRkoAAABXRUJQVlA4WAoAAAAQAAAAAAAAAAAAQUxQSAwAAAARBxAR/Q9ERP8DAABWUDggGAAAABQBAJ0BKgEAAQAAAP4AAA3AAP7mtQAAAA==';
		} else {
			addRootTag();
		}
	}(document));

## CSS文件中使用
	.classname {
		background-image: url(http://y.taofen8.com/xxx.jpg);
	}

	.webps .classname {
		background-image: url(http://y.taofen8.com/xxx.jpg@_.webp);
	}
	
## vue组件中使用
如果是静态图片的话，可以在computed中定义一个webp后缀变量来实现

	<img :src="'http://y.taofen8.com/xxx.jpg' + webpSuffix" />

其他动态图片的话，可以直接由服务端根据cookie，来决定下发哪种图片格式。

# retina兼容
前端应该都了解在retina屏下应该使用@2x或者@3x等倍率的图片，才能保证图片的清晰度。但是为了切图方便，部分公司都会统一在切图阶段，切出@2x的图片，不管浏览设备是retina屏还是普通屏，一律都使用@2x的图片。

这样做有两个坏处，一是@2x的图片在非retina屏下会出现downsampled现象，虽然不会影响清晰度，但是会缺少一些锐利度。二是@2x的图片相比较@1x的图片，前者体积大于后者，这也就造成了流量的浪费以及影响页面打开性能。

所以正确的处理方法是，针对retina屏和普通屏，采用不同尺寸的图片。图片裁剪已经在图片服务器上实现。在考虑retina时也需要加上webp的兼容，两者一起作用会大大减少图片的尺寸。
