---
title: css 点滴
date: 2018-01-26 17:40:43
tags: css
---
**transition**

> 为页面元素添加过滤效果

~~~
div
{
	width:100px;
	transition: width 2s;
	-moz-transition: width 2s; /* Firefox 4 */
	-webkit-transition: width 2s; /* Safari 和 Chrome */
	-o-transition: width 2s; /* Opera */
}
~~~

> transition 属性是一个简写属性，用于设置四个过渡属性：

1 transition-property
2 transition-duration
3 transition-timing-function
4 transition-delay

|值|说明|
|----|----|
|transition-property|规定设置过渡效果的 CSS 属性的名称。|
|transition-duration|规定完成过渡效果需要多少秒或毫秒。|
|transition-timing-function|规定速度效果的速度曲线。|
|transition-delay|定义过渡效果何时开始。|