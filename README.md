简介
====

这是用来使用 [谷歌开发者协议](https://chromedevtools.github.io/devtools-protocol/) 的类，特点是用纤程把异步回调转为同步调用

目录
===

* [依赖库](#依赖库)
* [类方法](#类方法)
    * [open](#open)
    * [connect](#connect)
    * [waitEvent](#waitEvent)
    * [run](#run)
	* [on](#on)
	* [setTimeout](#setTimeout)
* [示例](#示例)
	* [简单模拟点击](#简单模拟点击)
	* [获取节点内容](#获取节点内容)
	* [拦截指定资源类型的请求，例如不请求图片资源](#拦截指定资源类型的请求，例如不请求图片资源)

依赖库
=====

* [HP-Socket-aardio](https://github.com/btx638/HP-Socket-aardio)

类属性
=====

类方法
=====

open
----

**syntax** *open(chromePath, headless, userDataDir, port, disableGpu)*

|Argument |type|default|Optional| Description |
|:---------|:--:|:-----:|:-------|:------------|
|chromePath|string||*|谷歌浏览器路径|
|headless|boolean|false|*|以无头模式运行|
|userDataDir|string|\chrome.remote.userdata|*|用户目录|
|port|number||*|调试端口|
|disableGpu|boolean|true|*|禁用GPU|

[回到主题](#类方法)

run
----

**语法** *run(task, onTaskBegin, onTaskEnd, ...)*

[回到主题](#类方法)

connect
-------

**syntax** *connect()*

[回到主题](#类方法)

waitEvent
----

[回到主题](#类方法)

类属性
=====

示例 
====
* 运行前请把谷歌浏览器更新到最新版本

简单模拟点击
------------

![运行动画](https://raw.githubusercontent.com/btx638/dp/master/aaz/chrome/dp/example/1.gif)

````javascript
import win.ui;
/*DSG{{*/
var winform = win.form(text="aardio form";right=385;bottom=140)
winform.add(
button={cls="button";text="运行";left=142;top=43;right=221;bottom=83;z=1}
)
/*}}*/

io.open()

import aaz.chrome.dp;

var cdp, err = aaz.chrome.dp()
if(!cdp){
    winform.msgboxErr(err);
	return ; 
}
cdp.timeout = 20000;

var task = function(){
 	// 打开浏览器
	var ok, err = cdp.open()
    if(!ok){
         io.print("打开浏览器失败", err);
    	return ; 
    }
	
	// 连接浏览器
    var ok, err = cdp.connect();
    if(!ok){
        io.print("连接浏览器失败", err);
    	return ; 
    }

	// 订阅 Page 事件	
	var ok, err = cdp.Page.enable();
    if(!ok){
        io.print("订阅 Page 事件失败", err);
    	return ; 
    }

	// 打开网址
	var ok, err = cdp.Page.navigate(
		url = "https://www.so.com";
	)
    if(!ok){
        io.print("打开网址失败", err)
    	return ; 
    }
	
	// 等待页面加载完成
	var ok, err = cdp.waitEvent( "Page.loadEventFired" );
    if(!ok){
        io.print("等待页面加载完成失败", err);
    	return ; 
    }

	// 执行代码
	var ok, err = cdp.Runtime.evaluate(
		expression = /**
       document.querySelector("#input").value = "aardio";
       document.querySelector("#search-button").click();
		**/
	)

}

winform.button.oncommand = function(id,event){
	cdp.run(task)
}

winform.show();
win.loopMessage();
````

[Back to TOC](#Examples)

获取节点内容
------------


![运行动画](https://raw.githubusercontent.com/btx638/dp/master/aaz/chrome/dp/example/3.gif)

````javascript
import win.ui;
/*DSG{{*/
var winform = win.form(text="aardio form";right=663;bottom=383)
winform.add(
button={cls="button";text="运行";left=552;top=320;right=650;bottom=348;z=1};
edSelector={cls="edit";text=".items-area .item dl dt a";left=72;top=320;right=416;bottom=348;edge=1;z=6};
edUrl={cls="edit";text="https://www.cnbeta.com/";left=72;top=280;right=416;bottom=308;edge=1;z=4};
lvResult={cls="listview";left=9;top=7;right=652;bottom=271;edge=1;fullRow=1;z=2};
static={cls="static";text="url";left=11;top=286;right=64;bottom=310;align="right";transparent=1;z=3};
static2={cls="static";text="selector ";left=8;top=320;right=63;bottom=344;align="right";transparent=1;z=5}
)
/*}}*/

io.open()

import win.ui.statusbar;
import aaz.chrome.dp;

var cdp, err = aaz.chrome.dp()
if(!cdp){
    winform.msgboxErr(err);
	return ; 
}

// 设置每次请求的超时
cdp.setTimeout(9000);

var statusbar = win.ui.statusbar(winform);
statusbar.addItem("运行状态：", 70);
statusbar.addItem("就绪", -1);

winform.lvResult.insertColumn("序号",50);
winform.lvResult.insertColumn("标题",-1);

var uiLog = function(str){
	statusbar.setText(str, 2);
}

var currentTaskStepName;
var uiLogRunning = function(name){
    var str = string.format("正在%s...", name);
    currentTaskStepName = name;
	uiLog(str);	
}

var uiLogFail = function(name, err){
    name := currentTaskStepName;
    var str = string.format("%s失败，原因：%s", name, err);
	uiLog(str);
}

var uiResult = function(str){
	winform.lvResult.addItem({
		tostring(winform.lvResult.count+1);
		str
	});
}

var uiInit = function(){
	uiLog("任务开始", 2);
	winform.lvResult.clear();
}

var closeChrome = function(){
	return cdp.Browser.close(); 
}

var doFail = function(name, err){
    uiLogFail(name, err);
	closeChrome();
}

// 任务开始时触发
var onTaskBegin = function(){
	winform.button.disabled = true
	currentTaskStepName = null;
	uiInit();
}

// 任务结束时触发，可以接收任务函数的返回值
var onTaskEnd = function(result, err){
	winform.button.disabled = false
	
	if(!result){
		doFail(currentTaskStepName, err)
		return ; 
	}
	
	uiLog("任务完成")
	
	// 读取结果
	for(i=1;#result;1){
		uiResult(result[i]);
	}
}

var task = function(url, selector){
 	uiLogRunning("打开浏览器");
 	// 开启无头模式，没有界面，如果出现超时失败，可能是安装的谷歌浏览器版本太低，请更新到最新版本
	var ok, err = cdp.open(null, true);
    if(!ok){
    	return null, err; 
    }
	
	uiLogRunning("连接浏览器");
    var ok, err = cdp.connect();
    if(!ok){
    	return null, err; 
    }
    
	uiLogRunning("订阅 Page 事件");	
	var ok, err = cdp.Page.enable();
    if(!ok){
    	return null, err; 
    }
	
	// 打开网址
	uiLogRunning("打开网址");	
	var ok, err = cdp.Page.navigate(
		url = url;
	)
    if(!ok){
    	return null, err; 
    }
	
	uiLogRunning("等待页面加载完成");
	var ok, err = cdp.waitEvent("Page.loadEventFired");
    if(!ok){
    	return null, err; 
    }
 
	uiLogRunning("开启 runtime");
	var ok, err = cdp.Runtime.enable()
	if(!ok){
		return null, err; 
	}
	
	uiLogRunning("编译脚本");
	var ret, err = cdp.Runtime.compileScript(
		sourceURL = url;
		persistScript = true;
		expression = /**
		 var getInnerText = function(selector){
       		var doms = document.querySelectorAll(selector);
       		var ret = new Array();
       		for(var i=0;i<doms.length;i++){
           		ret.push(doms[i].innerText);
       		}
       		return ret; 
		 }
		**/
	)

	if(!ret){
    	return null, err; 
	}
	// 检查编译的结果
	// 发现错误
	if(ret.exceptionDetails){
    	return 
    		null, 
        	" 行号：" ++ ret.exceptionDetails.lineNumber ++
        	" 列号：" ++ ret.exceptionDetails.columnNumber ++ 
        	" 内容：" ++ ret.exceptionDetails.exception.description; 
	}
	
	
	uiLogRunning("运行脚本");
	var ret, err = cdp.Runtime.runScript(
		scriptId = 	ret.scriptId;
		returnByValue = true;
	)
	if(!ret){
		return null, err; 
	}
	
	uiLogRunning("对全局对象的表达式求值");
	var ret, err = cdp.Runtime.evaluate(
		expression = `getInnerText("` ++ selector ++ `")`;
		returnByValue = true;
	)
	if(!ret){
		return null, err; 
	}
	var result = ret.result;
	
	// 检测结果的类型
	// 看看有没有错误
	select(result.subtype) {
		case "error" {
			return null, result.description; 
		}
	}

	uiLogRunning("关闭浏览器");
	var ok, err = closeChrome();
	if(!ok){
		return null, err; 
	}
	
	// 返回结果
	return result.value; 
}

winform.button.oncommand = function(id,event){
	cdp.run(task, onTaskBegin,  onTaskEnd, winform.edUrl.text, winform.edSelector.text)
}



winform.show();
win.loopMessage();
return winform;
````

[返回目录](#示例)

拦截指定资源类型的请求，例如不请求图片资源
------------

![运行动画](https://raw.githubusercontent.com/btx638/dp/master/aaz/chrome/dp/example/4.gif)

````javascript
import win.ui;
/*DSG{{*/
var winform = win.form(text="aardio form";right=385;bottom=140)
winform.add(
btnRunTask={cls="button";text="运行";left=142;top=43;right=221;bottom=83;z=1}
)
/*}}*/

io.open()

import aaz.chrome.dp;

var cdp, err = aaz.chrome.dp()
if(!cdp){
    winform.msgboxErr(err);
	return ; 
}

// 接收事件
cdp.on("Fetch.requestPaused", function(params){
	// 请求类型为图片就中止请求
	if(params.resourceType == "Image"){
		cdp.Fetch.failRequest(
			requestId = params.requestId;
			errorReason = "Aborted";
		);
	}
	else {
		// 继续其他的请求
		cdp.Fetch.continueRequest(
			requestId = params.requestId;
		);
	}
})
	
// 任务函数
var task = function(){
 	// 打开浏览器
	var ok, err = cdp.open()
    if(!ok){
         io.print("打开浏览器失败", err);
    	return ; 
    }
	
	// 连接浏览器
    var ok, err = cdp.connect();
    if(!ok){
        io.print("连接浏览器失败", err);
    	return ; 
    }
    

    // 订阅 Fetch 事件，用于对网络请求的拦截
	 cdp.Fetch.enable()
 
	// 订阅 Page 事件	
	var ok, err = cdp.Page.enable();
    if(!ok){
        io.print("订阅 Page 事件失败", err);
    	return ; 
    }
	
	// 打开网址
	var ok, err = cdp.Page.navigate(
		url = "https://www.cnbeta.com/";
	)
    if(!ok){
        io.print("打开网址失败", err)
    	return ; 
    }
	
	// 等待页面加载完成
	var ok, err = cdp.waitEvent( "Page.loadEventFired" );
    if(!ok){
        io.print("等待页面加载完成失败", err);
    	return ; 
    }
}

var onTaskBegin = function(){
    winform.btnRunTask.disabled = true
}

var onTaskEnd = function(...){
    winform.btnRunTask.disabled = false
}

winform.btnRunTask.oncommand = function(id,event){
	cdp.run(task, onTaskBegin, onTaskEnd)
}

winform.show();
win.loopMessage();
````

[回到目录](#目录)