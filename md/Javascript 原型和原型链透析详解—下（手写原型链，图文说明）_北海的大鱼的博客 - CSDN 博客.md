> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/weixin_45745641/article/details/122456004?utm_medium=distribute.pc_feed_v2.none-task-blog-personrec_tag-7.pc_personrecdepth_1-utm_source=distribute.pc_feed_v2.none-task-blog-personrec_tag-7.pc_personrec)

前言
==

上一篇文章从 class 类入手写了个小案例，从而引入类型判断的方法 instanceof，带大家了解了什么是原型。这篇接着上篇去讲，给大家介绍重头戏—原型链，面试高频：如何手写原型链，对原型链的理解。  
没看过上集的建议点下方链接去瞅瞅，图文并茂，方便理解本文  
✿(。◕ᴗ◕。)✿  
[Javascript 原型和原型链透析详解—上](https://blog.csdn.net/weixin_45745641/article/details/122415646?spm=1001.2014.3001.5502)

原型链
---

上集文章中我们写了一个案例。在案例中定义了父类 People，子类 Student 继承父类，那么就有如下全等关系：  
People 的显示原型 prototype 全等 Student 显示原型的隐式原型

```
console.log( Student.prototype._proto__)
console.log( People.prototype )
console.log( People.prototype === Student.prototype._proto_
```

到这里大家是不是有点蒙，别急，接着上集中的图，继续来看这张图：  
下面的三个方框就是上集中的图，之前讲过了这里就不复述了。直接往下看，Student 这个原型有一个隐式原型__proto __ 指向了 People 的原型，里面有 eat 的方法。逻辑跟前三块的逻辑一致  
![](https://img-blog.csdnimg.cn/4ee42f1af5ce414bb17cff3d882e5534.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5YyX5rW355qE5aSn6bG8,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)  
到此，我们再来梳理一遍。访问 Jack.info() 时，Jack 本身没有这个属性，就会通过隐式原型__proto __ 访问到了 Student.prototype 中的 info() 函数。访问 Jack.eat() 时，自己没有，通过__proto __访问到 Student.prototype，还是没有，就会通过__proto __继续往上找。也就是说在每一级都会先访问自己的属性，如果没有，就会找它的隐式原型，一级一级往上，所以就形成了一个链

这一块搞清楚之后，继续往下看：（结合下面的图看）  
People 继承于 Object，所以 people 中有一个隐式原型指向 Object 的原型，object 原型里面的属性是 js 引擎自带的，有很多，这里只列举了一部分。所以如果要访问 Jack.hasOwnProperty 这个方法的话，就会层层向上找，一直找到 Object 的原型形成一个原型链。  
那么能不能顺着 Object 继续往上找呢？Object 的隐式原型永远指向 null，到此为止，无法再向上  
![](https://img-blog.csdnimg.cn/c545eb31af69470197ce364d78186d7a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5YyX5rW355qE5aSn6bG8,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)  
学到了这里，再回头来看上集讲过的 instanceof  
![](https://img-blog.csdnimg.cn/2af401c351094e9da724bf4b1d54cf5b.png#pic_center)  
instanceof 前面的变量会顺着隐式原型一层层往上找，如果能找到第二个变量（class）的显示原型，就返回 true，反之 false。根据这句话看看上图，大家应该就能够明白，instanceof 是基于原型链实现的

最后关于原型链，大家一定要把上面的图理解懂并且能够默写出来，还有这个 instanceof 原理，面试中经常会在里面考察你对原型链的理解和掌握程度~