# dp
[Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/) lib for aardio

#### Depend on [HP-Socket-aardio](https://github.com/btx638/HP-Socket-aardio)
#### example

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