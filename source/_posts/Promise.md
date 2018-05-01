title: "jQuery Deferred详解"
date: 2015-04-11 21:32:39
categories: 技术 #文章文类
tags: js  #文章标签，多于一项时用这种格式[]
description: Promise是一种模式，以同步操作的流程形式来操作异步事件，避免了层层嵌套，可以链式操作异步事件。
keywords: Promise javascript
---
### Promise
Promise是一种模式，以同步操作的流程形式来操作异步事件，用链式方式操作异步事件，避免了层层嵌套。

我们知道，在编写javascript异步代码时，callback（回调）是最最简单的机制，可是用这种机制的话必须牺牲控制流、异常处理和函数语义化为代价，甚至会让我们掉进出现callback嵌套的坑，而promise解决了这个问题。

Promise能解耦事件注册和事件处理的代码，在模块化开发中，使用Promise是个不错的选择。

### Deferred
在jquery中，jQuery开发团队设计了deferred对象，它遵守Common Promise/A 规范。
<!--more-->
#### 一、什么是deferred对象？
简单说，deferred对象就是jQuery的回调函数解决方案。在英语中，defer的意思是"延迟"，所以deferred对象的含义就是"延迟"到未来某个点再执行。

它解决了如何处理耗时操作的问题，对那些操作提供了更好的控制，以及统一的编程接口。它的主要功能，可以归结为四点。下面我们通过示例代码，一步步来学习

#### 二、deferred对象的状态
jQuery规定，deferred对象有三种执行状态----未完成（`pending`），已完成（`resolved`）和已失败（`rejected`）。如果执行状态是已完成，deferred对象立刻调用`done()`方法指定的回调函数；如果执行状态是已失败，调用`fail()`方法指定的回调函数；如果执行状态是未完成，则继续等待，或者调用`progress()`方法指定的回调函数（jQuery1.7版本添加）。

`deferred.resolve()`：将deferred对象的执行状态从未完成改为已完成，从而触发done()方法。
`deferred.reject()`：将deferred对象的执行状态从未完成改为已失败，从而触发fail()方法。

#### 三、deferred对象的promise()
`deferred.promise()`：deferred.promise()返回一个promise对象，是没有 resolve ,reject, progress , resolveWith, rejectWith , progressWith 这些可以改变状态的方法，你只能使用 done, then ,fail 等方法添加 handler 或者判断状态。

#### 四、解析jquery Deferred源码
下面是jquery的Deferred源码：
```javascript
Deferred: function( func ) {
    var tuples = [
            // action, add listener, listener list, final state
            [ "resolve", "done", jQuery.Callbacks("once memory"), "resolved" ],
            [ "reject", "fail", jQuery.Callbacks("once memory"), "rejected" ],
            [ "notify", "progress", jQuery.Callbacks("memory") ]
        ],
        //初始状态是pending
        state = "pending",
        //promise对象只能添加deferred对象状态改变的handle和判断状态
        promise = {
            state: function() {
                return state;
            },
            always: function() {
                deferred.done( arguments ).fail( arguments );
                return this;
            },
            /**返回一个新deferred对象的promise
            * fnDone 成功的回调函数
            * fnFail 失败的回调函数
            * fnProgress 表示进度的回调函数
            */
            then: function( /* fnDone, fnFail, fnProgress */ ) {
                var fns = arguments;
                return jQuery.Deferred(function( newDefer ) {
                    jQuery.each( tuples, function( i, tuple ) {
                        var fn = jQuery.isFunction( fns[ i ] ) && fns[ i ];
                        // deferred[ done | fail | progress ] for forwarding actions to newDefer
                        deferred[ tuple[1] ](function() {
                            var returned = fn && fn.apply( this, arguments );
                            if ( returned && jQuery.isFunction( returned.promise ) ) {
                                returned.promise()
                                    .done( newDefer.resolve )
                                    .fail( newDefer.reject )
                                    .progress( newDefer.notify );
                            } else {
                               //把fn函数的返回结果当作参数传给下一个对应处理函数
                                newDefer[ tuple[ 0 ] + "With" ]( this === promise ? newDefer.promise() : this, fn ? [ returned ] : arguments );
                            }
                        });
                    });
                    fns = null;
                }).promise();
            },
            // Get a promise for this deferred
            // If obj is provided, the promise aspect is added to the object
            promise: function( obj ) {
                return obj != null ? jQuery.extend( obj, promise ) : promise;
            }
        },
        deferred = {};

    // Keep pipe for back-compat
    promise.pipe = promise.then;

    // Add list-specific methods
    jQuery.each( tuples, function( i, tuple ) {
        var list = tuple[ 2 ],
            stateString = tuple[ 3 ];

        // promise[ done | fail | progress ] = list.add
        promise[ tuple[1] ] = list.add;

        // Handle state
        if ( stateString ) {
            list.add(function() {
                // state = [ resolved | rejected ]
                state = stateString;

                // [ reject_list | resolve_list ].disable; progress_list.lock
            }, tuples[ i ^ 1 ][ 2 ].disable, tuples[ 2 ][ 2 ].lock );
        }

        // deferred[ resolve | reject | notify ]
        deferred[ tuple[0] ] = function() {
            deferred[ tuple[0] + "With" ]( this === deferred ? promise : this, arguments );
            return this;
        };
        deferred[ tuple[0] + "With" ] = list.fireWith;
    });

    // Make the deferred a promise
    promise.promise( deferred );

    // 如果传递了一个函数func，则把deferred对象当作参数传到func中，并且函数func内部this指向deferred
    if ( func ) {
        func.call( deferred, deferred );
    }

    // All done!
    return deferred;
}
```
由上面jquery的源码可以看出`deferred.promise()`返回的promise对象是函数内部的局部变量，它提供的函数能操作deferred对象的属性，运用了`闭包`技术。

#### 五、构建延迟对象的最佳实践
**下面是运用Deferred构建延迟对象的最佳实践**，deferred对象在函数内部，不会被外部污染或者改变其状态，返回promise对象只关注状态发生改变之后的handle
~~~javascript
var wait = function(){
    var dtd = $.Deferred(); //在函数内部，新建一个Deferred对象
　　var tasks = function(){
　　　　alert("执行完毕！");
　　　　dtd.resolve(); // 改变Deferred对象的执行状态
　　};

　　setTimeout(tasks,5000);
　　return dtd.promise(); // 返回promise对象
};
var promise = wait();
promise
    .done(function(){ alert("哈哈，成功了！"); })
    .fail(function(){ alert("出错啦！"); });
~~~

**tips：查看js源码是学习前端技术最快的途径之一**