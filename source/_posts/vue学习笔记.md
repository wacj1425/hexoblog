---
title: vue学习笔记
date: 2018-02-01 11:33:28
tags: [vue,前端,js]
categories: 学习
---
## 基础

### 声明式渲染

1. 模板语法：`{{message}}`
2. 属性绑定： v-bind:title="message"

### 条件与循环

1. 条件判断 v-if="seen"
2. 循环 v-for="todo in todos"


## vue_cli下的开发

###关于 mounted 如下：
> 这个问题困惑了好长时间！

```
<template>
  <h1>{{name}}</h1>
</template>

<script>
export default {
  name: 'index',
  data () {
    return {
      name: '测试数据as'
    }
  },
  mounted () {
    this.name = 'name在页面加载完成之后被重载'
  }
}
</script>
```