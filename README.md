# datetimepicker

兼容vue2.0 不兼容vue1.0

目录说明
src 核心源码
demo 一个基于vue2.0的使用实例

组件开发依赖两个工具类：utils.js和NodeList.js
使用时要引入


在线实例
https://fgopan.github.io/datetimepicker/

组件属性说明

| Property      | Type          | Default | Required | Description |
| ------------- |:-------------:| -------:|---------:|------------:|
| value|String||TRUE|数值|
| hasTime|Boolean|false|时分|
| format|String|mm/dd/yyyy||时间格式|
| disabled-days-of-week|Array|[]| |星期几禁选,数值有效范围0~6|
| start|String|''||开始时间限制|
| end|String|''||结束时间限制|
| width|String|160px||组件宽度|
| clear-button|Boolean|false| |是否有清除按钮|
| lang|String|navigator.language| |语言，默认中文|
| placeholder|String|''| |提示|
| btn-text|String|ok| |按钮文本|
| btn-class|String|''| |按钮样式|
| btn-style|Object|''| |按钮样式 {'color':'red'}|

| Methods|Args|Required|Description|
| ------ |:--:|-------:|----------:|
| on-data-change|val|TRUE|监听value变化，vue2.0不支持value.async的写法，代替方案|
