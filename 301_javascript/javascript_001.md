# ios微信H5网页没有声音的解决方法

# 一、问题描述

在做微信网页抽奖开发时，点击抽奖需要添加音效， 正常情况下，直接调用audio标签的play方法即可，但是在ios微信端不起作用。

# 二、解决方法

通过`WeixinJSBridge`调用play方法，如下：

```js

// lotteryAudio为audio标签的id
var oAudio = document.getElementById('lotteryAudio');
if (window.WeixinJSBridge) {
	WeixinJSBridge.invoke('getNetworkType', {}, function (e) {
		oAudio.play();
	}, false);
} else {
	document.addEventListener("WeixinJSBridgeReady", function () {
		WeixinJSBridge.invoke('getNetworkType', {}, function (e) {
			oAudio.play();
		});
    }, false);
}
oAudio.play();

```



