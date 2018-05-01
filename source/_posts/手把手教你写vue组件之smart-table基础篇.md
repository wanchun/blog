title: "手把手教你写vue组件之smart-table基础篇"
date: 2016-05-15 00:16:30
categories: 技术 #文章文类
tags: [vue, js]  #文章标签，多于一项时用这种格式[]
description: Table是最常用展示数据的方式之一，一个好用的table组件能让开发事半功倍。
keywords: vue js 组件
---
最近用vue开发了一个管理系统类的产品。产品需求里面有许多list查询界面，需要一个table组件来搞定它们。

### 第一步 分析
产品里面的table要满足如下功能：
* 固定的头部
* 按照头部列内容排列一行一行的展示数据
* table需要实现分页查询
* table可以根据条件查询内容
* 我们开发肯定知道源数据可能是1、2这种key，需要用数据字典把key转化为说明文字
* 特殊的操作列，内容是按钮，并且能触发动作

仔细想想，其实还有:
* 设置列的宽度
* 设置列的text-align
* 点击列的title排序table
* 其他

### 第二步 动手做
想再多，不去动手做肯定不行。切忌考虑太多，不需要考虑清楚怎么实现去所有功能，只需要按照你的思路往下做就好。往往做的过程中会有更好的idea，困难总能解决的。
开发业务的同时给table组件添加子功能，最终实现了上面所有功能。（ps: 代码在内网，不方便拿出来）


### 第三步 反思
虽然这个table组件能简单轻松实现业务需求，但是它还是有一些问题：
* 数据字典过滤器和table过滤器在filter目录，功能上没有高内聚，跟工程目录结构有耦合
* ajax请求配置跟api模块是有耦合的
* 配置是通过rule对象传入table组件，很长很长的一个json对象
<!--more-->

这样的table组件怎么拿出来开源！它不是我理想中的table组件。我想要一个高内聚完全独立的table组件，我想要一个语义化的table组件。在我眼里table组件应该长这个样子：
```html
    <mytable>
        <column data-key="a" name="col-a" ></column>
        <column data-key="b" name="col-b" ></column>
        <column data-key="b" name="col-c" ></column>
        <column name="操作"  action="{text:'删除',func:'callback'}"></column>
    </mytable>
```
抛弃使用json对象而使用column结构配置table的话面临一个问题，如何把column结构转化为json对象，并且传递给table组件？
思考了一番，有一个办法，使用Vue的[Slot功能](http://cn.vuejs.org/guide/components.html#使用-Slot-分发内容)将column组件分发到table组件里面，然后在table组件通过遍历子组件column拿到它的配置。我们来看一下column的实现：
```html
    <template>
        <!---这个span必须要，不然子组件就不加载了--->
        <span style="display: none"></span>
    </template>
    <script>
        import Vue from 'vue';
        var util = Vue.util;
        function parseText(str){
            if(str.startsWith("{") || str.startsWith("[")){
                //应该有更高效的，献丑
                str = str.replace(/'/g, "\"");
                str = str.replace(/(\s?\{\s?)(\S)/g, "$1"+"\""+"$2");
                str = str.replace(/(\s?,\s?)(\S)/g, "$1"+"\""+"$2");
                str = str.replace(/(\S\s?):(\s?\S)/g, "$1"+"\":"+"$2");
                str = str.replace(/,"\{/g, ",{");
                str = str.replace(/""/g, "\"");
                str = str.replace(/\s/g, "");
                try{
                    str = JSON.parse(str)
                }catch(e){
                }
            }
            return str;
        }
        export default {
            props: {
                dataKey: {
                    type: String
                },
                name: {
                    type: String,
                    required: true
                },
                align: {
                    type: String,
                    default: 'left'
                },
                filter: [String, Array],
                width: String,
                action: [String, Array, Object]
            },
            data: function () {
                return {}
            },
            ready: function () {
                //把{key:1}变成object
                var filter = this.filter;
                if(filter && !util.isObject(filter)){
                    this.filter = parseText(filter);
                }
                var action = this.action;
                if(action && !util.isObject(action)){
                    this.action = parseText(action);
                    if(util.isObject(this.action)){
                        this.action = [this.action];
                    }
                }
            }
        }
    </script>
    <style>
    </style>
```
在table组件中拿到column配置：
```javascript
    //这是table组件的ready事件钩子
    ready: function () {
        var _this = this;
        this.$children.forEach(function (child) {
            //把column的配置整合到rule中
            var obj = {};
            for(var p in child._props){
                obj[p] = child[p]
            }
            _this.rule.push(obj)
        })
    },
```
语义化的问题完美解决，哈哈！


**在开发过程又遇到一个问题**：我希望column中prop项的值可以是string，可以是对象，可以是数组，还可以是类对象String。至于为什么，稍后说明配置项action和filter就知道了。
解决这个问题的思路是把类对象string转化了标准JSON格式的string，然后parse。解决方案如下：
```javascript
function parseText(str){
    if(str.startsWith("{") || str.startsWith("[")){
        //应该有更高效的，献丑
        str = str.replace(/'/g, "\"");
        str = str.replace(/(\s?\{\s?)(\S)/g, "$1"+"\""+"$2");
        str = str.replace(/(\s?,\s?)(\S)/g, "$1"+"\""+"$2");
        str = str.replace(/(\S\s?):(\s?\S)/g, "$1"+"\":"+"$2");
        str = str.replace(/,"\{/g, ",{");
        str = str.replace(/""/g, "\"");
        str = str.replace(/\s/g, "");
        try{
            str = JSON.parse(str)
        }catch(e){
        }
    }
    return str;
}
```

### 第四步 完成初版
由于精力有限，先开发一个基础版，包含如下功能：
* 固定的头部
* 按照头部列内容排列一行一行的展示数据
* 用数据字典把key转化为说明文字
* 特殊的操作列，内容是按钮，并且能触发动作
* 设置列的宽度
* 设置列的text-align

下面是table组件源代码：
```html
<template>
    <div>
        <table>
            <thead>
            <tr>
                <th v-for="item in rule"
                    class="col_{{$index+1}}"
                    :style="getStyle(item)"
                    @click="thColClick(item, $event)">
                    {{item.name}}
                </th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(rowIndex, trData) in data | table rule" class="row_{{rowIndex+1}}" @click="bodyTrClick(data[rowIndex], $event)">
                <td v-for="(colIndex, tdData) in trData"
                    track-by="$index"
                    :style="getStyle(rule[colIndex])"
                    class="col_{{colIndex+1}}">
                    {{render(tdData, rule[colIndex])}}
                    <template v-if="tdData === null && rule[colIndex].action">
                        <span v-for="actionItem in rule[colIndex].action" @click.stop="fireAction(actionItem, data[rowIndex], $event)">
                            {{actionItem.text}}
                        </span>
                    </template>
                </td>
            </tr>
            </tbody>
        </table>
        <slot></slot>
    </div>
</template>
<script>
    import Vue from 'vue'
    var util = Vue.util;
    export default {
        props: ['data'],
        data: function () {
            return {
                rule: []
            }
        },
        filters: {
            table: function (data, rule) {
                var arr = data.slice(0);
                var _arr = [];
                arr.forEach(function (trDate) {
                    var __arr = [];
                    for (var i = 0; i < rule.length; i++) {
                        __arr.push(trDate[rule[i].dataKey] || null)
                    }
                    _arr.push(__arr);
                });
                return _arr
            }
        },
        ready: function () {
            var _this = this;
            this.$children.forEach(function (child) {
                //把column的配置整合到rule中
                var obj = {};
                for(var p in child._props){
                    obj[p] = child[p]
                }
                _this.rule.push(obj)
            })
        },
        methods: {
            //渲染列 可以根据类型渲染不同的样式，比如说渲染普通文本，渲染数字，渲染过滤后的文本，可以自定义渲染的td
            render: function (tdData, rule) {
                //如果filter存在
                if(rule.filter){
                    var filter = rule.filter;
                    if(typeof filter == "string"){
                        tdData = (this[filter] && this[filter](tdData)) || tdData;
                    }else if(util.isArray(filter)){
                        var theOne = filter.filter(function (o) {
                            return o.key == tdData;
                        });
                        if(theOne.length > 0){
                            tdData = theOne[0].value
                        }
                    }
                }
                return tdData
            },
            //设置样式
            getStyle: function (col) {
                return {
                    "text-align" : col.align,
                    "width" : col.width
                }
            },
            //点击th列
            thColClick: function (item, event) {
                this.$dispatch("th-col-click", item);
            },
            //点击内容行
            bodyTrClick: function (rowData, event) {
                this.$dispatch("body-tr-click", rowData)
            },
            //触发action动作
            fireAction: function (action, rowData, event) {
                if(typeof action.func == "string"){
                    this.$parent[action.func].call(this.$parent, rowData)
                }
                if(typeof action.func == "function"){
                    action.func(rowData)
                }
            },
            //其实是一个过滤器
            status: function (value) {
                var arr = {
                    "1" : "有效",
                    "2" : "无效"
                };
                return arr[value]
            }
        }
    }
</script>
<style scoped>
    table {
        width: 100%;
        max-width: 100%;
        border: 1px solid #d7d7d7;
        border-spacing: 0;
        border-collapse: collapse;
    }
    tr{
        border-bottom: 1px solid #e4e4e4;
    }
    td, th{
        padding: 10px 20px;
    }
    th{
        background-color: #e8e8e8;
    }
</style>
```

我称呼它为vue-smart-table

### 配置详解
```html
    <mytable :data="data" @th-col-click="thColClick" @body-tr-click="trClick">
        <column slot data-key="a" name="col-a" align="center" width="130px" filter="status"></column>
        <column slot data-key="b" name="col-b" width="130px" :filter="haha"></column>
        <column slot data-key="b" name="col-c" filter="[{ key : 1,value : '星期一'},{ key : 2,value : '星期二'}]"></column>
        <column slot name="操作"  align="center" width="230px" action="{text:'删除',func:'callback'}"></column>
    </mytable>
```
**配置项：**
`data-key` ： 数据的key，定义这一列显示数据对象哪一个属性的值
`name`     ： 定义列的title名称
`align`    ： 定义列文字排列方向，可选center、left（默认）、right
`width`    ： 定义列的宽度
`filter`   ： 定义列的过滤器，可以是过滤器函数的名称，可以是数组[{key:1,value:'哈哈'}]，也可以是类对象数组"[{key:1,value:'哈哈'}]"
```javascript
    //这里是table的父组件，使用table的地方
    data: function(){
        return {
            haha : []
        }
    }
    ready: function () {
        var _this = this;
        setTimeout(function () {
            _this.haha.push({
                key : 1,
                value : "星期一"
            })
            _this.haha.push({
                key : 2,
                value : "星期二"
            })
        },1000)
    }
```
filter的值可以是异步拿到的，拿到之后会执行转换动作，非常好用！！！vue实在是好用！！
`action`    ： 定义特殊的操作列，可以是对象和数组，也可以是类对象string。类对象string的话回调属性func的值是父组件的methods中的函数名。
`data`      ： 源数据list

**事件：**
`th-col-click`  ：点击列的头部触发，事件参数是 列的配置对象
`body-tr-click` ：点击table的body触发，事件参数是 行的原始数据值
`func`          ：特殊操作触发， 事件参数是 行的原始数据值