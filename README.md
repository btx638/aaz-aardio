Name
====

a little lib for usting[Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/) in [aardio](http://www.aardio.com/)

Table of Contents
=================

* [Name](#name)
* [Dependent](#Dependent)
* [Example](#example)
    * [简单模拟点击](#简单模拟点击)
	* [获取节点属性](#获取节点属性)
	* [用脚本获取节点内容](#用脚本获取节点内容)

Dependent
=========

* [HP-Socket-aardio](https://github.com/btx638/HP-Socket-aardio)

Example
=======

简单模拟点击
------------

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

var cdp = aaz.chrome.dp();
// 接收事件
cdp.onChromeEvent = function(method, params){
	io.print("事件名字：", method);
	io.print("事件内容：", table.tostring(params));
}

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
        io.print("打开网址失败", err);
    	return ; 
    }

	// 运行脚本
	var ok, err = cdp.Runtime.evaluate(
		expression = /**
       document.querySelector("#input").value = "aardio";
       document.querySelector("#search-button").click();
		**/
	)

}

winform.button.oncommand = function(id,event){
	io.print(cdp.run(task));
}

winform.show();
win.loopMessage();
````

获取节点属性
------------

````javascript
import win.ui;
/*DSG{{*/
var winform = win.form(text="aardio form";right=239;bottom=143)
winform.add(
button={cls="button";text="开始";left=37;top=36;right=203;bottom=115;z=1}
)
/*}}*/

io.open()
import console
import aaz.chrome.dp;

var cdp = aaz.chrome.dp()

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
		url = "https://www.htmlayout.cn/";
	)
    if(!ok){
        io.print("打开网址失败", err)
    	return ; 
    }
	
	// 等待页面加载完成
	var ok, err = cdp.waitEvent( "Page.loadEventFired" );
    if(!ok){
        io.print("打开网址失败", err);
    	return ; 
    }
 
	// 获取 document 节点
	var document, err = cdp.DOM.getDocument()
    if(!document){
        io.print("获取 document 节失败", err);
    	return ; 
    }
    
    // 读取帖子列表
	var doms, err = cdp.DOM.querySelectorAll(
		nodeId = document.root.nodeId;
		selector = ".home-box-list .post-list .item-content h2 a";
	)
    if(!doms){
        io.print("获取节点失败", err);
    	return ; 
    }
    
    // 读取帖子的超链接
	for(i=1;#doms.nodeIds;1){
		var ret, err = cdp.DOM.getAttributes(
			nodeId = doms.nodeIds[i]
		);
		if(!ret){
			io.print( "读取帖子的超链接 出错：", err )
			return ; 
		}

		for(j=1;#ret.attributes;2){
			var k = ret.attributes[j];
			var v = ret.attributes[j+1];
			io.print(k, v);
		}
	}
}

winform.button.oncommand = function(id,event){
	io.print(cdp.run(task));
}

winform.show();
win.loopMessage();
return winform;
````

用脚本获取节点内容
------------------

![运行动画](https://raw.githubusercontent.com/btx638/dp/master/aaz/chrome/dp/example/3.gif)

````javascript
import win.ui;
/*DSG{{*/
var winform = win.form(text="aardio form";right=629;bottom=362)
winform.add(
button={cls="button";text="运行";left=528;top=290;right=617;bottom=326;z=1};
lvResult={cls="listview";left=9;top=6;right=621;bottom=278;edge=1;fullRow=1;z=2}
)
/*}}*/

import win.ui.statusbar;
import aaz.chrome.dp;

var cdp = aaz.chrome.dp()

var statusbar = win.ui.statusbar(winform);
statusbar.addItem("运行状态：", 70);
statusbar.addItem("", -1);

winform.lvResult.insertColumn("序号",50);
winform.lvResult.insertColumn("标题",-1);

var uiLog = function(str){
	statusbar.setText(str, 2);
}

var uiResult = function(str){
	winform.lvResult.addItem({
		tostring(winform.lvResult.count+1);
		str
	})
}

var uiInit = function(){
	uiLog("任务开始", 2);
	winform.lvResult.clear();
}

var task = function(){
    uiInit();
 	
 	uiLog("正在打开浏览器...");
	var ok, err = cdp.open(true); // 打开浏览器并开启 headless 模式，即不显示界面
    if(!ok){
        uiLog("打开浏览器失败，原因: " ++ err);
    	return ; 
    }
	
	uiLog("正在连接浏览器...");
    var ok, err = cdp.connect();
    if(!ok){
        uiLog("连接浏览器失败，原因: " ++ err);
    	return ; 
    }
    
	uiLog("正在订阅 Page 事件...");	
	var ok, err = cdp.Page.enable();
    if(!ok){
        uiLog("订阅 Page 事件失败，原因:" ++ err);	
    	return ; 
    }
	
	// 打开网址
	uiLog("正在打开网址 https://www.htmlayout.cn/ ...");	
	var ok, err = cdp.Page.navigate(
		url = "https://www.htmlayout.cn/";
	)
    if(!ok){
        uiLog("打开网址 https://www.htmlayout.cn/ ，原因:" ++ err);
    	return ; 
    }
	
	uiLog("正在等待页面加载完成...");
	var ok, err = cdp.waitEvent( "Page.loadEventFired" );
    if(!ok){
        uiLog("等待页面加载完成失败，原因: " ++ err);
    	return ; 
    }
 	
	uiLog("正在开启 runtime...");
	var ok, err = cdp.Runtime.enable()
	if(!ok){
		uiLog("开启 runtime 失败，原因：" ++ err);
	}
	
	uiLog("正在编译脚本...");
	var script, err = cdp.Runtime.compileScript(
		sourceURL = "https://www.htmlayout.cn/";
		persistScript = true;
		expression = /**
		 var getTitles = function(){
       		var doms = document.querySelectorAll(".home-box-list .post-list .item-content h2 a");
       		var ret = new Array();
       		for(var i=0;i<doms.length;i++){
           		ret.push(doms[i].innerText);
       		}
       		return ret; 
		 }
		**/
	)
	if(!script){
        uiLog("编译脚本失败，原因: " ++ err);
    	return ; 
	}
	
	uiLog("正在运行脚本...");
	var ret, err = cdp.Runtime.runScript(
		scriptId = 	script.scriptId;
		returnByValue = true;
	)
	if(!ret){
		uiLog("运行脚本失败，原因:" ++ err);
		return ; 
	}
	
	uiLog("对全局对象的表达式求值...");
	var ret, err = cdp.Runtime.evaluate(
		expression = "getTitles()";
		returnByValue = true;
	)
	if(!ret){
		uiLog("对全局对象的表达式求值失败，原因:" ++ err);
		return ; 
	}
	
	// 读取结果
	for(i=1;#ret.result.value;1){
		uiResult(ret.result.value[i])
	}
	
	// 关必浏览器
	uiLog("正在关必浏览器...");
	var ok, err = cdp.Browser.close()
	if(!ok){
		uiLog("关必浏览器，原因:" ++ err);
		return ; 
	}
	
	uiLog("任务完成");
}

winform.button.oncommand = function(id,event){
	cdp.run(task)
}

winform.show();
win.loopMessage();
return winform;
````