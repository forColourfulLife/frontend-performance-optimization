# 图片流量优化
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
