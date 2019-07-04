## 代码
项目地址：[一个vue日历组件](https://github.com/lihao336991/vue-calendar)

## 需求
最近业务需求需要一个日历组件，要求可切换周视图和月视图，并且可以显示日程安排及事件标注。

看了下网上已有的日历组件，[fullcalendar](https://github.com/fullcalendar/fullcalendar)插件star还挺多的，文档也挺全面的，但是使用起来不够简便，和我司的应用场景也有些许出入，考虑再三决定还是重新封装一个日历组件。

## 组件封装
整体思路如下：

1、将涉及日期时间计算的逻辑封装到组件内部，对外暴露change方法，可获取当前日历展示的时间段；

2、另外除接收一些配置信息外，还可以接收事件集合，用于渲染到对应时间格子上，相应的点击事件及日期改变也已经暴露出来，可以根据需要自行封装修改。
```
<full-calendar :events="fcEvents" @eventClick="fetchEvent" lang="zh" @change="changeDateRange"></full-calendar>
```
3、除此之外，考虑到不同业务逻辑都可能用到日历组件，事件卡片的展示没有封装进去，通过slot暴露出来进行自由编码（参考Element的table元素）。
```
<!--子组件-->
<slot name="fc-body-card"></slot>

<!--父组件-->
<template slot="fc-body-card">
    <!------>
</template>
```

## 效果图


![](https://user-gold-cdn.xitu.io/2019/7/4/16bbc439fe089f45?w=1516&h=643&f=png&s=55411)

[演示地址: 一个vue日历组件——vue-calendar](http://calendar.yisujin.cn/)
