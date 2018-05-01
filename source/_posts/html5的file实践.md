title: "html5的file实践"
date: 2016-08-20 14:51:36
categories: 技术 #文章文类
tags: [html5]  #文章标签，多于一项时用这种格式[]
description: html5的file实践，新api的运用
keywords: 版本管理工具
---
最近工作上遇到一些处理file的相关需求，比如说有上传文件、预览图片、下载图片和音频文件、播放音频文件等。为了解决上面的需求使用了html5的File、Blob、FileReader、XMLHttpRequest 2.0、FormData等API。在此做个笔记整理下，提升学习效果。

## 一. File和Blob
>一个Blob对象就是一个包含有只读原始数据的类文件对象。Blob对象中的数据并不一定得是JavaScript中的原生形式。File接口基于Blob，继承了Blob的功能,并且扩展支持了用户计算机上的本地文件。
创建Blob对象的方法有几种，可以调用Blob构造函数，还可以使用一个已有Blob对象上的slice()方法切出另一个Blob对象，还可以调用canvas对象上的toBlob方法。

以上为MDN上官方口吻的解释。实际上，Blob是计算机界通用术语之一，全称写作：BLOB (binary large object)，表示二进制大对象。MySql/Oracle数据库中，就有一种Blob类型，专门存放二进制数据。

### 1.1 Blob构造函数
```javascript
Blob Blob(
  [optional] Array parts,
  [optional] BlobPropertyBag properties
)
```
**parts**
　　一个数组，包含了将要添加到Blob对象中的数据。数组元素可以是任意多个的`ArrayBuffer`，`ArrayBufferView (typed array)`， `Blob`，或者 `DOMString`对象。
**properties**
　　一个对象，设置Blob对象的一些属性

比如说下面的代码可以创一个xml格式的数据：
```javascript
var aFileParts = ['<a id="a"><b id="b">hey!</b></a>'];
var oMyBlob = new Blob(aFileParts, { "type" : "text/xml" }); // the blob
```
### 1.2 canvas.toBlob方法
<!--more-->
>HTMLCanvasElement.toBlob() 方法创造Blob对象，用以展示canvas上的图片；这个图片文件可以被缓存或保存到本地，由用户代理端自行决定。如不特别指明，图片的类型默认为 image/png，分辨率为96dpi。

使用canvas.toBlob可以将canvas图像转换为文件，下面是代码：

```javascript
var canvas = document.getElementById("canvas");
canvas.toBlob(function(blob) {
  var newImg = document.createElement("img"),
      url = URL.createObjectURL(blob);
  newImg.onload = function() {
    // no longer need to read the blob so it's revoked
    URL.revokeObjectURL(url);
  };
  newImg.src = url;
  document.body.appendChild(newImg);
});
```
### 1.3 XMLHttpRequest.responseType = Blob
XMLHttpRequest2.0中设置XMLHttpRequest.responseType = Blob，接受到响应时XMLHttpRequest.response是一个blob对象。
```javascript
function readBlobURL(src, callback) {
    var xhr = new XMLHttpRequest();
    var blob;
    xhr.open("GET", src, true);
    xhr.responseType = 'blob';
    xhr.onload = function (e) {
        if (e.target.status == 200)
            return callback(e.target.response); // e.target.response也就是请求的返回就是Blob对象
    }
    xhr.send();
    return blob;
}
```
比如说在项目中我需要播放服务器上的`.amr`和`.spx`音频文件，html5的audio标签是不支持这两种格式的，怎么办呢？整理下思路，有专门的js库可以解码amr或者spx文件为原始音频数据，然后用AudioContext的api去播放原始音频数据。然而解码函数的入参是ArrayBuffer或者BinaryString格式的音频数据，所以先得把音频文件弄成ArrayBuffer或者BinaryString。刚好FileReader可以把一个Blob对象读成先ArrayBuffer和BinaryString。

噔噔噔噔噔蹬蹬噔，XMLHttpRequest.responseType = Blob闪亮登场！！！直接从服务器把音频文件下载为Blob对象。需求完美解决，可以安心回家陪女朋友~~

### 1.4 File
File继承于Blob对象，所以本质上是一个只读原始数据的类文件对象。不过呢，file对象多了name、size、type、lastModified属性。
```
<input type="file" id="harrywan">
<script>
    document.getElementById('harrywan').addEventListener('change', function (evt) {
        var file = evt.target.files[0];
        console.log(file)
    }, false);
</script>
```
下图是选择一个图片之后console打印的结果：
![](/imgs/file.png)
上图可以清晰的看到name、size、type属性。file对象的构造器是File，File对象继承于Blob。

## 二. FileReader
>使用FileReader对象，web应用程序可以异步的读取存储在用户计算机上的文件(或者原始数据缓冲)内容，可以使用File对象或者Blob对象来指定所要处理的文件或数据。

FileReader有五个方法：
```javascript
void abort();
//开始读取指定的Blob对象或File对象中的内容. 当读取操作完成时,readyState属性的值会成为DONE,如果设置了onloadend事件处理程序,则调用之.同时,result属性中将包含一个ArrayBuffer对象以表示所读取文件的内容.
void readAsArrayBuffer(in Blob blob);
//result属性中将包含所读取文件的原始二进制数据.
void readAsBinaryString(in Blob blob);
//result属性中将包含一个data: URL格式的字符串以表示所读取文件的内容.
void readAsDataURL(in Blob blob);
//result属性中将包含一个字符串以表示所读取的文件内容.
void readAsText(in Blob blob, [optional] in DOMString encoding);
```
`readAsArrayBuffer`返回的是arrayBuffer，`readAsBinaryString`返回的是二进制string，可以通过下面的方法转换：
```javascript
function ab2str(buf) {
  return String.fromCharCode.apply(null, new Uint16Array(buf));
}

function str2ab(str) {
  var buf = new ArrayBuffer(str.length*2); // 2 bytes for each char
  var bufView = new Uint16Array(buf);
  for (var i=0, strLen=str.length; i<strLen; i++) {
    bufView[i] = str.charCodeAt(i);
  }
  return buf;
}
```
BinaryString通过window.btoa()加密，从而组装成data规范的string。

`readAsDataURL`可以很方便的实现图片预览：
```
//假设file是一个File对象
var image = new Image();
var file = e.target.files[0];
var fileReader = new FileReader();
fileReader.onload = function (oFREvent) {
  var data = oFREvent.target.result; //data就是data规范的base64数据
  image.src = data;
};
fileReader.readAsDataURL(file);
```

## 三. FormData
>XMLHttpRequest Level 2添加了一个新的接口FormData.利用FormData对象,我们可以通过JavaScript用一些键值对来模拟一系列表单控件,我们还可以使用XMLHttpRequest的send()方法来异步的提交这个"表单".比起普通的ajax,使用FormData的最大优点就是我们可以异步上传一个二进制文件.

### 2.1 FormData构造函数
```javascript
new FormData (form? : HTMLFormElement)
```
**form** (可选)：一个HTML表单元素,可以包含任何形式的表单控件,包括文件输入框。
### 2.2 append()
```javascript
void append(DOMString name, Blob value, optional DOMString filename);
void append(DOMString name, DOMString value);
```
给当前FormData对象添加一个键/值对，值可以是Blob和DOMString。
### 2.3 上传文件
下面是使用FormData的例子：
```javascript
var oMyForm = new FormData();

oMyForm.append("username", "Groucho");
oMyForm.append("accountnum", 123456); // 数字123456被立即转换成字符串"123456"

// fileInputElement中已经包含了用户所选择的文件
oMyForm.append("userfile", fileInputElement.files[0]);

var oFileBody = '<a id="a"><b id="b">hey!</b></a>'; // Blob对象包含的文件内容
var oBlob = new Blob([oFileBody], { type: "text/xml"});

oMyForm.append("webmasterfile", oBlob);

var oReq = new XMLHttpRequest();
oReq.open("POST", "http://foo.com/submitform.php");
oReq.send(oMyForm);
```
给我的感觉，用FormData就像我们提交form表单，而且还是异步的！有了FormData完全不必要去伪造一个隐藏的form实现文件的上传。ajax默认的content-type是application/x-www-form-urlencoded，使用FormData之后，content-type是multipart/form-data。

用FormData可以非常简单的实现文件上传。并且FormData兼容性也不错：
![](/imgs/formdata.png)
### 2.4 伪造上传按钮
一个文件filele类型的input长下面这个样子，很丑吧。
![](/imgs/fileInput.png)
自从跟设计师配合之后，选择文件要么被设计成一个icon，要么是一个按钮的样式，美美的。以前，我会放置一个隐藏的input，然后用控制div盖住input，点击div时，相当于点击了input触发文件选择的效果。但是效果不太好，点击域和手势效果不友好。

从Gecko 2.0 (Firefox 4 / Thunderbird 3.3 / SeaMonkey 2.1)开始，你可以隐藏掉默认的的文件输入框`<input>`元素，使用自定义的界面来充当打开文件选择对话框的按钮。实现起来很简单，你只需要使用样式display:none把原本的文件输入框隐藏掉，然后在需要的时候调用它的click()方法就行了。

考虑下面的HTML：
```
<input type="file" id="fileElem" multiple accept="image/*" style="display：none" onchange="handleFiles(this.files)">
<a href="#" id="fileSelect">Select some files</a>
var fileSelect = document.getElementById("fileSelect"),
  fileElem = document.getElementById("fileElem");
fileSelect.addEventListener("click", function (e) {
  if (fileElem) {
    fileElem.click();
  }
  e.preventDefault();
}, false);
```
这样，你就能任意改变这个文件选择按钮的样式了。设计师，来吧，随便虐。

### 参考链接
[https://developer.mozilla.org/zh-CN/docs/Using_files_from_web_applications](https://developer.mozilla.org/zh-CN/docs/Using_files_from_web_applications)
[http://www.zhangxinxu.com/wordpress/2013/10/understand-domstring-document-formdata-blob-file-arraybuffer/](http://www.zhangxinxu.com/wordpress/2013/10/understand-domstring-document-formdata-blob-file-arraybuffer/)
[https://developer.mozilla.org/zh-CN/docs/Web/API/File](https://developer.mozilla.org/zh-CN/docs/Web/API/File)
[https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)
[https://developer.mozilla.org/zh-CN/docs/Web/API/FormData](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)
[https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)

