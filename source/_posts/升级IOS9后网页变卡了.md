title: "升级IOS9后网页变卡了"
date: 2015-09-30 11:38:36
tags: [移动开发,css,html]
description: 刚刚参与开发了一个移动H5项目，遇到一些坑
keywords: webApp html5 css3
---

        <p>9月份，苹果发布了IOS9系统。本来没觉得此事会我的工作造成影响，实际上给我带来了两个坑。</p>
<h3 id="一、动画变卡了">一、动画变卡了</h3><p>当时我负责一个H5项目<a href="https://f.fangdd.com/ddh/" target="_blank" rel="external">多多惠</a>。突然有一天我们的产品妹子问我为什么多多惠首页选择城市的弹出框特别特别慢？经过多番核查发现只有IOS9才有此问题，然后猜想是动画时间异常。看下面的CSS代码：</p>
<pre><code><span class="at_rule">@<span class="keyword">keyframes</span> fadeIn </span>{
    0% <span class="rules">{<span class="rule"><span class="attribute">opacity</span>:<span class="value"> <span class="number">0</span></span></span>;}</span>
    100% <span class="rules">{<span class="rule"><span class="attribute">opacity</span>:<span class="value"> <span class="number">1</span></span></span>;}</span>
}
<span class="class">.fadeIn</span> <span class="rules">{
    <span class="rule"><span class="attribute">animation-name</span>:<span class="value"> fadeIn</span></span>;
    <span class="rule"><span class="attribute">animation-duration</span>:<span class="value"> <span class="number">300ms</span></span></span>;
    <span class="rule"><span class="attribute">animation-fill-mode</span>:<span class="value"> both</span></span>;
}</span>
</code></pre><p>谁能想到<code>300ms</code>会导致bug呢？把<code>300ms</code>改为<code>.3s</code>就OK了</p>
<h3 id="二、在app中的webview中触发原生登录动作之后，返回之前的webview地址，传参失败！">二、在app中的webview中触发原生登录动作之后，返回之前的webview地址，传参失败！</h3>