title: "手把手教你写vue组件之sort-table进阶篇"
date: 2016-05-17 22:42:38
categories: 技术 #文章文类
tags: [vue, js]  #文章标签，多于一项时用这种格式[]
description: Table是最常用展示数据的方式之一，一个好用的table组件能让开发事半功倍。有各种各样特别的需求table，是搞一个大而全的，还是细分组合？
keywords: vue js 组件
---
接上一篇[手把手教你写vue组件之smart-table基础篇](/2016/05/15/手把手教你写vue组件之smart-table基础篇/)

上一篇的smart-table封装了table的几个基础功能。咋们还需要给它实现排序、ajax接口、分页呢，so继续改造smart-table。但是一味的往上面添加功能，a、b、c、d...吗？**NO!NO!NO!**那样做会弄出一个庞然大物。在以前，我会找一些开源的jq插件来快速开发页面。有些插件真的很强大而且好用，体积也大，它的大部分其他功能我用不上。这让天秤座的我很纠结，我不喜欢多余的代码。

自己怎么能让自己难受呢？我认为理想中的组件应该这样：

以功能的粒度用定义组件，让组件可以自由组合。比方说有组件a孙悟空、组件b金箍棒、组件c筋斗云。想要孙悟空就直接用a，想要拿着金箍棒的孙悟空就用a+b，想要踩着七彩祥云的孙悟空就用a+c，想要最帅的孙悟空就用a+b+c。
<!--more-->
恰好，vue提供了[`extends`](http://cn.vuejs.org/api/#extends)和[`Mixin`](http://cn.vuejs.org/guide/mixins.html#基础)两种功能，能让组件自由组合。下面是我实现的vue-sort-table的源码：
```html
//SortTable.vue
<template>
    <div>
        <table>
            <thead>
            <tr>
                <th v-for="item in rule"
                    :style="getStyle(item)"
                    class="col_{{$index+1}}"
                    :class="{sort:sortKey==item.dataKey, up:order>=0, down:order<0 }"
                    @click="thColClick(item, $event)">
                    {{item.name}}
                </th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="(rowIndex, trData) in data | orderBy sortKey order | table rule"  class="row_{{rowIndex+1}}"
                @click="bodyTrClick(data[rowIndex], $event)">
                <td v-for="(colIndex, tdData) in trData"
                    track-by="$index"
                    :style="getStyle(rule[colIndex])"
                    class="col_{{colIndex+1}}">
                    {{render(tdData, rule[colIndex])}}
                    <template v-if="tdData === null && rule[colIndex].action">
                        <span v-for="actionItem in rule[colIndex].action"
                              @click.stop="fireAction(actionItem, data[rowIndex], $event)">
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
    import table from './Table'
    export default {
        extends: table,
//        mixins: [table],
        data: function () {
          return {
              order : 1,
              sortKey : ""
          }
        },
        methods: {
            thColClick: function (rule, event) {
                if(rule.sort === true || rule.sort === "true"){
                    if(this.sortKey == rule.dataKey){
                        this.order = this.order * -1;
                    }else{
                        this.sortKey = rule.dataKey;
                        this.order = 1;
                    }
                }
            }
        }
    }
</script>
<style scoped>
    th.sort{
    }
    th.sort.down:before{
        display: inline-block;
        content: "";
        width: 0;
        height: 0;
        border-left: 5px solid transparent;
        border-right: 5px solid transparent;
        border-top: 10px solid black;
    }
    th.sort.up:before{
        display: inline-block;
        content: "";
        width: 0;
        height: 0;
        border-left: 5px solid transparent;
        border-right: 5px solid transparent;
        border-bottom: 10px solid black;
    }
</style>
```
代码里面只添加了处理sort的逻辑，看着多么清爽。在业务逻辑里面这么用：
```html
<sort-table :data="data" @th-col-click="thColClick" @body-tr-click="trClick">
    <column slot data-key="a" name="col-a" align="center" width="130px" filter="status"></column>
    <column slot data-key="b" name="col-b" width="130px" sort="true"></column>
    <column slot data-key="c" name="col-c" filter="[{ key : 1,value : '星期一'},{ key : 2,value : '星期二'}]"></column>
    <column slot name="操作"  align="center" width="230px" action="{text:'删除',func:'callback'}"></column>
</sort-table>
```
跟上一篇的smart-table相比只是标签名称变了，多了一个配置`sort="true"`
![](/imgs/sort.gif)