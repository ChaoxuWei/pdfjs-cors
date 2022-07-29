start file: **pdfjs-develop/web/test.html**

启动文件：**pdfjs-develop/web/test.html**

**Add annotation functions for pdf.js:** https://demos.libertynlp.com/#/pdfjs-annotation

**给pdf.js增加批注功能:** https://demos.libertynlp.com/#/pdfjs-annotation


**中文版在后半部分**

## English

This is the solutioin to throughly solve the CORS issue of pdf.js. 

Worked live demo and source code: [https://demos.libertynlp.com/#/pdfjs-cors](https://demos.libertynlp.com/#/pdfjs-cors)

Steps:
#### 1.1 In Valid PDF.js CORS
To invalid **PDF.js** CORS，disable the following code in viewer.js。
```
// original code
      if (origin !== viewerOrigin && protocol !== "blob:") {
        throw new Error("file origin does not match viewer's");
      }

// disabled them
      // if (origin !== viewerOrigin && protocol !== "blob:") {
      //   throw new Error("file origin does not match viewer's");
      // }
```

#### 1.2 Bypass browser CORS
1. Disable the following code in viewer.js, then override the functions for loading PDF file  named webViewerLoad and Run.

```
// inactivate follow original code in viewer.js

//first place
function webViewerLoad() {
	var config = getViewerConfiguration();
	window.PDFViewerApplication = pdfjsWebApp.PDFViewerApplication;
	window.PDFViewerApplicationOptions = pdfjsWebAppOptions.AppOptions;
	var event = document.createEvent("CustomEvent");
	event.initCustomEvent("webviewerloaded", true, true, {});
	document.dispatchEvent(event);
	pdfjsWebApp.PDFViewerApplication.run(config);
}

//second place
if (document.readyState === "interactive" || document.readyState === "complete") {
	webViewerLoad();
} else {
	document.addEventListener("DOMContentLoaded", webViewerLoad, true);
}

//third place
run: function run(config) {
	this.initialize(config).then(webViewerInitialized);
},
```

2. **rewrite webViewerLoad and run fuctions**

```
// rewrite webViewerLoad function
window.webViewerLoad = function webViewerLoad(fileUrl) {
	var config = getViewerConfiguration();
	window.PDFViewerApplication = pdfjsWebApp.PDFViewerApplication;
	window.PDFViewerApplicationOptions = pdfjsWebAppOptions.AppOptions;
	var event = document.createEvent('CustomEvent');
	event.initCustomEvent('webviewerloaded', true, true, {});
	document.dispatchEvent(event);

	if (fileUrl) {
		config.defaultUrl = fileUrl;
	}
	pdfjsWebApp.PDFViewerApplication.run(config);
}

//rewrite run function
//modeify for browser CORS
run: function run(config) {
	var _that = this;
	//add judgement
	if (config.defaultUrl) {
		_app_options.AppOptions.set('defaultUrl', config.defaultUrl)
	}

	_that.initialize(config).then(function() {
		webViewerInitialized()
	});
},
```

#### 1.3 Call new webViewerLoad

Add a function in viewer.html to call the modified webViewerLoad function when viewer.html onloading.

```
< script type = "text/javascript" >
	window.onload = function() {
		var pdfUrl = "https://heritagesciencejournal.springeropen.com/track/pdf/10.1186/s40494-021-00620-2.pdf";
		webViewerLoad(pdfUrl);
	}
</script>
```


## 中文版

要彻底解决 **PDF.js** 的跨域问题，让 **PDF.js** 可以从 url 加载文档，需要解决 **PDF.js** 本身和浏览器的双重跨域问题。

正常运行的Demo和源码：***<https://demos.libertynlp.com/#/pdfjs-cors>***
源码是我已经完成所有跨域设置的 **PDF.js** 代码，下载后导入你的项目中即可从 url 动态加载pdf。

#### 1.1 禁用PDF.js跨域

要禁用 **PDF.js** CORS，需要在 ***viewer.js*** 文档中将下面一段代码注释掉，让它失效。
```
// 原代码
      if (origin !== viewerOrigin && protocol !== "blob:") {
        throw new Error("file origin does not match viewer's");
      }

// 注释掉上方代码
      // if (origin !== viewerOrigin && protocol !== "blob:") {
      //   throw new Error("file origin does not match viewer's");
      // }
```

#### 1.2 绕过浏览器跨域

要解决浏览器 URL 文件跨域的问题，可以通过后端服务器将PDF 文件转换成流文件的方式返回给 **PDF.js**，不过这里我们不讨论这样的策略，而是讨论如何只在前端解决这个问题。**按照以下步骤可以解决问题。**

1. 在 ***viewer.js*** 中注释掉以下三处代码
```
// inactivate follow original code in viewer.js

//first place
function webViewerLoad() {
	var config = getViewerConfiguration();
	window.PDFViewerApplication = pdfjsWebApp.PDFViewerApplication;
	window.PDFViewerApplicationOptions = pdfjsWebAppOptions.AppOptions;
	var event = document.createEvent("CustomEvent");
	event.initCustomEvent("webviewerloaded", true, true, {});
	document.dispatchEvent(event);
	pdfjsWebApp.PDFViewerApplication.run(config);
}

//second place
if (document.readyState === "interactive" || document.readyState === "complete") {
	webViewerLoad();
} else {
	document.addEventListener("DOMContentLoaded", webViewerLoad, true);
}

//third place
run: function run(config) {
	this.initialize(config).then(webViewerInitialized);
},
```

2. **重写 webViewerLoad 和 run 函数**
```
// 重写 webViewerLoad 函数
window.webViewerLoad = function webViewerLoad(fileUrl) {
	var config = getViewerConfiguration();
	window.PDFViewerApplication = pdfjsWebApp.PDFViewerApplication;
	window.PDFViewerApplicationOptions = pdfjsWebAppOptions.AppOptions;
	var event = document.createEvent('CustomEvent');
	event.initCustomEvent('webviewerloaded', true, true, {});
	document.dispatchEvent(event);

	if (fileUrl) {
		config.defaultUrl = fileUrl;
	}
	pdfjsWebApp.PDFViewerApplication.run(config);
}

//rewrite run function
//modeify for browser CORS
run: function run(config) {
	var _that = this;
	//add judgement
	if (config.defaultUrl) {
		_app_options.AppOptions.set('defaultUrl', config.defaultUrl)
	}

	_that.initialize(config).then(function() {
		webViewerInitialized()
	});
},
```

#### 1.3 调用以上修改

在 ***viewer.html*** 中新增一个函数，目的是在加载页面时调用修改过的 ***webViewerLoad*** 函数。
```
< script type = "text/javascript" >
	window.onload = function() {
		var pdfUrl = "https://heritagesciencejournal.springeropen.com/track/pdf/10.1186/s40494-021-00620-2.pdf";
		webViewerLoad(pdfUrl);
	}
</script>
```

## 2. 从URL动态加载PDF

修改 **viewer.html** 中的函数，根据 **viewer.html** 所在 **iframe** 标签 **src** 中携带的 **PDF url** 加载文件。
```
<script type = "text/javascript" >
	window.onload = function() {
		var all_href = location.href;
		var file_id = all_href.split('?')[1];
		var pdfUrl = file_id.split('=')[1];
		// var pdfUrl='https://fireflycos.libertynlp.com/firefly-static/new_shouce.pdf';
		webViewerLoad(pdfUrl);
	}
</script>
```
