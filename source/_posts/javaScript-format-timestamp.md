---
title: JavaScript 时间戳格式转换
date: 2018-10-31 23:11:00 +0800
updated: 2018-10-31 23:11:00 +0800
comments: false
---
## 问题
将时间戳转换为指定格式的字符串。
## 方案
```js
function formatDate(time,format='YY-MM-DD hh:mm:ss'){
	var date = new Date(time);
	var year = date.getFullYear(),
	    month = date.getMonth()+1,//月份是从0开始的
	    day = date.getDate(),
	    hour = date.getHours(),
	    min = date.getMinutes(),
	    sec = date.getSeconds();
	var preArr = Array.apply(null,Array(10)).map(function(elem, index) {
		return '0'+index;
	});//开个长度为10的数组 格式为 00 01 02 03
	var newTime = format.replace(/YY/g,year)
                        .replace(/MM/g,preArr[month]||month)
                        .replace(/DD/g,preArr[day]||day)
                        .replace(/hh/g,preArr[hour]||hour)
                        .replace(/mm/g,preArr[min]||min)
                        .replace(/ss/g,preArr[sec]||sec);
	return newTime;         
}
```
## 注意！
此方案存在 IE 兼容性问题。可以使用 `arguments[i]` 为参数设置默认值，`i` 为参数的位置，第一个为 0。
```js
function example(a, b){
	var a = arguments[0] || 1;//设置第一个参数的默认值为1
	var b = arguments[1] || 2;//设置第二个参数的默认值为2
	···
}
```
