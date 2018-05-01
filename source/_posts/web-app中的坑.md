title: "web app开发中不得不说的坑"
date: 2015-04-09 10:40:40
categories: 技术 #文章文类
tags: [移动开发,js,css,html]  #文章标签，多于一项时用这种格式[]
description: 刚刚参与开发了一个移动H5项目，遇到一些坑
keywords: webApp html5
---
我要说点什么呢？
### html
#### meta的viewport
在移动Web开发中，在head头部遇到最常见的问题，就是viewport的设置
``` bash
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1.0, minimum-scale=1.0, minimal-ui, user-scalable=0">
```
最佳实践：
- 一般情况下，在所有无线页面的头部，都要加上此viewport的设置，如果不加上此数值，会造成在某些webkit游览器中，游览器会根据自身的某些判断，自行做放大、缩小等，造成页面无法正常访问，特别是某些app中嵌入了webkit游览器来进行访问的时候， 会出现以上所说的情况，因此为了保证你说设计的网页在所有手机中显示保持一致，加上此设置
- viewport中的设置数值一般不需要进行修改，因为现在的数值已经满足了绝大多数项目，当然会出现在非常特殊的页面里，需要用户进行手动缩放的操作，不过如果修改了数值，需要在不同的手机上进行详细的测试，否则会有你预期外的事情发生。

<!--more-->

### css

#### type为number的input框
type为number的input框在部分手机浏览器上输入框右边有spinner,以下CSS可以屏蔽：
``` bash
input[type=number]::-webkit-inner-spin-button,
input[type=number]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
```

#### -webkit-tap-highlight-color
这个属性只用于iOS和android等webkit内核浏览器。当你点击一个链接或者通过Javascript定义的可点击元素的时候，它就会出现一个半透明的灰色背景，非常丑！
``` bash
html{
  -webkit-tap-highlight-color:rgba(255,255,255,0);  //可以同时屏蔽ios和android下点击元素时出现的阴影。
  -webkit-appearance:none  //可以同时屏蔽输入框怪异的内阴影。
}
```

#### flex
下面是flex在浏览器中支持情况：

| Chrome        | Safari           | firefox  | Opera | Android | iOS |
| ------------- |:-------------:| -----:| -----:| -----:| -----:|
| 21+ (modern) 20- (old) | 3.1+ (old) | 2-21 (old) 22+ (new) | 12.1+ (modern) |　2.1+ (old) | 3.2+ (old) |

- flex有三个版本，分别对应于display: box、display: flexbox、display: flex，不同浏览器版本使用的名称不一样,使用时需要考虑降级处理。
- 目前移动浏览器中使用比较多的有safari、UC、 QQ浏览器（微信）、andriod原生浏览器、chrome，UC和QQ浏览器不支持flex。所以虽然好用，但是少用，兼容性不好。

#### :nth-child伪类
在PC的各大浏览器中，除了IE8及以下不支持，其他浏览器支持良好。但是在手机浏览器中IOS4不支持nth-child，鉴于目前iPhone4S和iPhone4用户量还挺多，所以需要慎重使用。
添加class控制是解决这个简单但不优雅的方案。


### javascript

#### click的300ms延迟响应
1. andriod4.0+浏览器内核层面已经处理此bug
2. andriod4.0以下设置viewport的width=device-width 可以避免此问题
3. IOS上的safari只有通过js库来解决
3.1. 使用zetpo.js的tap事件，但是存在点透bug

    ``` bash
    $(elem).on("tap",function(){
    })
    ```

  3.2. 使用fastclick.js

    ``` bash
    FastClick.attach(document.body);  //在dom加载完毕
    ```

#### withCredentials和DOM Exception 11
_INVALID_STATE_ERR code 11_
试图使用一个不可用的对象。这种错误的抛出通常是因为某些内部原因，方法无法实现特定的操作。

_withCredentials_
服务器配置Access-Control-Allow-Origin，然后在浏览器中配置withCredentials=true,则可以实现跨域发送cookie和验证信息。然后开发移动web app遇到下面这个坑：

``` bash
var xhr = new XMLHttpRequest();
xhr.withCredentials = true; //支持跨域发送cookies
xhr.open("POST", "http://weibo.com/demo/b/index.php", true);
xhr.send();
```

``` bash
var xhr = new XMLHttpRequest();
xhr.open("POST", "http://weibo.com/demo/b/index.php", true);
xhr.withCredentials = true; //支持跨域发送cookies
xhr.send();
```

   唯一的差异就是xhr.withCredentials = true的位置。但就是这个差别，导致第一段代码无法顺利在手机端运行，并报INVALID_STATE_ERR: DOM Exception 11这个错！在IOS4和IOS5的UC浏览器、Safari、Chrome，进行CORS访问均会报这个错，Android4.0原生浏览器，也无法正常CORS（没有测试2.3和2.2）。
   而在桌面版浏览器下，两段代码都可以顺利运行！**所以，以后设置withCredentials属性时，一定要在open方法之后！**

很不幸，移动开发中流行的zepto.js中招了
``` bash
if (settings.xhrFields) for (name in settings.xhrFields) xhr[name] = settings.xhrFields[name]

var async = 'async' in settings ? settings.async : true
xhr.open(settings.type, settings.url, async, settings.username, settings.password)
```
把源码中第一行换到第四行之后就行