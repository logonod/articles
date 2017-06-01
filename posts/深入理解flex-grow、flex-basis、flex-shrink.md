> 布局的传统解决方案基于 盒模型 ，依赖 display + position + float ，flex 为布局带来了新的时代，我们再也不用局限于 float 和 position ，特别是在移动端，我们可以利用 flex 轻松实现以往 float 和 psition 很难实现的布局。 本文主要讲解flex的三个子属性：flex-grow、flex-basis、flex-shrink

如果你对 flex 还没有接触过，或者概念模糊，推荐阮老师的 [flex 布局教程](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

#### flex 布局发生在父容器和子容器之间

采用 flex 布局的元素，称为 flex 容器（flex container）。它的所有子元素自动成为容器成员。成员根据 flex 的规则瓜分容器的**剩余空间**。

#### 什么是剩余空间？

具备 flex 环境的父容器，通常是有一条水平的主轴和一条垂直的交叉轴。剩余空间就是父容器在主轴的方向上还有多少可用的空间。比如看下面这段html结构：

```html
<style>
.container {
  width: 500px;
  display: flex;
  display: -webkit-flex;
}
</style>
<div class="container">
  <span class="B1"></span>
  <span class="B2"></span>
  <span class="B3"></span>
</div>
```

container 就是父容器，B1 B2 B3 就是子容器。剩余空间就是：500px - B1.width - B2.width - B3.width。嗯就是这么简单！

#### flex-grow

知道了剩余空间的概念，首先来看一下 flex-grow。上面那个例子，现在我们再假设 B1、B2、B3 的 width 是100px，那么剩余空间就是 500-100*3=200px。 知道了剩余空间有什么用呢？这个时候 flex-grow 就该出场了，假如我们这个时候对 B1 设置 `flex-grow: 1;`，那么我们会发现，B1 把 B2 和 B3 都挤到右边了，也就是说剩余的200px空间都被 B1 占据了，所以此时 B1 的 width 比实际设置的值要大。

所以这里 flex-grow 的意思已经很明显了，就是索取父容器的剩余空间，默认值是0，就是三个子容器都不索取剩余空间。

但是当 B1 设置了 `flex-grow: 1;` 的时候，剩余空间就会被分成一份，然后都给了 B1。 如果此时B2设置了 `flex-grow: 2;`，那么说明 B2 也参与到瓜分剩余空间中来，并且他是占据了剩余空间中的2份，那么此时父容器就会把剩余空间分为3份，然后1份给到B1，2份给到B2。

#### flex-basis 

flex-basis 的作用也就是 width 的替代品。 如果子容器设置了 flex-basis 或者 width，那么在分配空间之前，他们会先跟父容器预约这么多的空间，然后剩下的才是归入到剩余空间，然后父容器再把剩余空间分配给设置了flex-grow 的容器。 如果同时设置 flex-basis 和 width，flex-basis 的优先级比 width 高。有一点需要注意，如果flex-basis 和 width 其中有一个是 `auto`，那么另外一个非 `auto` 的属性优先级会更高。

#### flex-shrink

该属性来设置当父元素的宽度小于所有子元素的宽度的和时（即子元素会超出父元素），子元素如何缩小自己的宽度。
flex-shrink 的默认值为1，当父元素的宽度小于所有子元素的宽度的和时，子元素的宽度会减小。值越大，减小的越厉害。如果值为0，表示始终不减小。

如上面的例子 B1、B2、B3的 width 都为 200px 时，子元素的宽度和超过父元素的 500px，如果 B1 设置了 `flex-shrink: 0;` ，则 B1 的大小为 200px，B2 设置 `flex-shrink: 3;`，B3 设置 `flex-shrink: 2;`，则 B2、B3的缩小比例为 **3：2**。下面就是求解方程的过程了，我们可以设置 B2 的缩小比为 3a，B3 的缩小比为 2a，则有等式 **200px * (1 - 3a) + 200px * (1- 2a) = 500px - 200px** ，计算得到 a = 0.1，所以 B2 的 width 为 140px，B3 的width 为 160px。

#### 总结

1. 如果父容器空间不够，就使用压缩 flex-shrink ，否则使用扩张 flex-grow
2. 如果你不希望某个容器在任何时候都不被压缩，那设置 flex-shrink: 0
3. 如果子容器的的 flex-basis 设置为0，那么计算剩余空间的时候将不会为子容器预留空间