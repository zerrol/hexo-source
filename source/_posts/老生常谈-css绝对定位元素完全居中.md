---
title: 老生常谈-css绝对定位元素完全居中
categories: css
tag: 
     - css
     - html    
     - 绝对定位与相对定位
     - 老生常谈
     - 最佳实践
---

# 通过计算实现居中
如果父元素的宽度是写死的话，对于绝对定位元素我们最直接的做法就是粗暴的计算元素的宽度值和容器的宽度之间的关系例如，我们先设置父容器的`css`属性如下：
```css
.father-container {
  height: 300px;
  width: 300px;
  background: #eee;
  margin: 10px auto;  /* 父元素block的水平居中*/
  position: relative;
}
```
我们可以注意到两个地方，第一是`wdith`属性，这里我设置了父容器的宽度是300px。然后是父元素的定位方式我们设置了`relative`,这个是为了限制子元素进行绝对定位以后依然被约束在父元素之内。子元素我们设置的属性如下：
```css 
.box {
  height: 100px;  /*为了更方便辨认，我设置成为了而一个方块样式*/
  width: 100px;
  background: #222;
  position: absolute;
  
  /*Centering Method*/ 
  left: 100px; 
}
```
实现的效果的话大致是这个样子的：
![居中样式](/img/17-6/22-1.jpg)
如果是宽度width是流的形式的话呢，例如：
```css
.father-container {
  height: 300px;
  width: 70%;    /*注意这里，父元素的宽度变成了百分号*/
  background: #eee;
  margin: 10px auto;  /*父元素block的水平居中*/
  position: relative;
}
```
在我们的父元素容器宽度是百分号的时候，说明他是一个自适应的宽度，这个时候我们大致计算模式是这样的：
![居中样式2](/img/17-6/22-2.jpg)
这个时候我们的子元素就对应该这样去实现：
```css
.box {
  height: 100px;
  width: 100px;  /*这里子元素的宽度依然是写死的*/
  background: #222;
  position: absolute;
  
  /*Centering Method*/ 
  margin: 0px 0 0 -50px;  /*需要通过-50px去修正居中*/
  left: 50%;
}
```
同理如果我们需要子元素也是自适应的百分比，那么通过修个修正的`margin`即可：
```css
.box {
  height: 100px;
  width: 80%;  /*这里子元素的宽度是自适应百分比*/
  background: #222;
  position: absolute;
  
  /*Centering Method*/ 
  margin: 0px 0 0 -40%;  /*需要通过margin左移40%去修正*/
  left: 50%;
}
```
实现的效果如下：
![居中计算](/img/17-6/22-3.png)

注意事项：这样实现最大的问题自然是需要去计算宽度的关系，在计算时我们需要特别注意的是无论是第一种直接通过`left`去调整位置，还是第二个方法里面通过`margin`去修正时，我们必须将任何会对元素宽度造成影响的因素考虑进去，例如：`border`、`padding`，我们计算时应该使用的是元素在渲染后的宽度，而不单单只是我们设置的`width`值。

这里其实就是通过最为简单计算来进行来实现了子元素的水平居中，这种方式的限制也是比较大。我们必须要保证子元素和父元素的宽度都是限定死的，不然的话就需要通过js代码去实时计算宽度来实现水平居中，就显得有些不够优雅了。

当然同理的话也可以通过这个方式实现**垂直居中**。

# 使用`transform`避免计算居中
上面的方法我需要去求得需要居中的元素的总宽度然后才能够通过`margin`进行修正，修正的方式就是通过减去元素自身宽度或高度的一半来是使元素处于完全居中的一个位置。

那么如果我们能够直接就获取到元素移动她位移自身宽度的一半不就可以实现了吗？没错，这个时候我们可以用到`css3`中加入`transform`属性，我们可以通过这个属性获取到获取自身的宽度，即便是没有设定显示的宽度或者高度，是通过内部元素撑起的也可以正确的获取到。（关于`transform`的具体特性和用法我们下次再讲）。

```css
.father-container {
  height: 300px;
  width: 70%;    /*注意这里，父元素的宽度变成了百分号*/
  background: #eee;
  margin: 10px auto;  /*父元素block的水平居中*/
  position: relative;
}
```
因此对应与上面同样的父级容器，我们居中就简单多了：

```css
.box {
  height: 100px;
  width: 80%;  /*这里子元素的宽度是自适应百分比*/
  background: #222;
  position: absolute;
  
  /*Centering Method*/ 
  left: 50%;
  top：50%;   /* 垂直居中 */
  transform: translate(-50%, -50%);  /*直接移动自身的高度和宽度的一半*/
}
```

优点：不需要再通过计算就可以实现完全居中，即便元素是**无宽度**的元素，`child-box`的宽度是由内部的文本撑起的情况下，也可以通过这个方式实现完全居中。

缺点：兼容性不好。因为`transform`本来就是`css3`的特性，IE9(-ms-),IE10+以及其他现代浏览器才支持。中国盛行的IE8浏览器被忽略是有些不适宜的（手机web开发可忽略）。

因此权衡之后对于**有宽度**的绝对定位元素居中不妨考虑下面这种方式。

# 使用`margin: auto`实现完全居中

对于绝对定位的元素还有一种十分巧妙的办法可以实现相对于父元素的完成**完全居中**，这里完全居中的意思是可以同时完成**水平居中**和**垂直居中**。首先我们来看一下实现方式再来研究一下原理。

```css
.father-container {
  height: 300px;
  width: 70%;   
  background: #eee;
  margin: 10px auto;  /*父元素block的水平居中*/
  position: relative;
}

/*未居中的子元素*/
.child-box {
  height: 100px;
  width: 100px;  
  background: #222;
  position: absolute;
}
```
为居中的表现就是一个正常的显示的绝对定位块：
![absolute-block](/img/17-6/27-1.png)

如果要使`.child-box`实现完全居中的话我们只需要做两件事情，第一个设置位置属性利用absolute流的特性进行将元素平铺开：`top: 0; bottom: 0; left: 0; right: 0;`，这个时候元素就已经被我们拉伸开了，下面再设置`margin: auto`就发现我们的`child-box`完成了完全居中。
```css
.child-box {
  height: 100px;
  width: 100px;  
  background: #222;
  position: absolute;
  
  /*Centering Method*/
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  margin: auto; /*key*/
}
```
效果如图：
![absolute-center](/img/17-6/27-2.png)
是不是很神奇的发现我们的绝对定位元素已经实现完全居中了！这种实现居中的方式对于绝对定位的元素的来说优点很多，既方便不需要计算，而且兼容性也很好，对于**有宽度**的元素来说，可以说是绝对定位的居中的最佳实践了。

为什么仅对于**有宽度**的绝对的定位元素来说呢，首先我们需要知道通过这个方式可以实现绝对定位元素完全居中的**原理**：

使用上面方法进行居中的时候，我们首先设置了`top: 0; bottom: 0; left: 0; right: 0;`这个东西，对于绝对定位元素来说当同时设置了**对立**的定位方向属性值的时候，就会触发绝对定位元素的流体特性。例如设置`top: 10px; bottom: 10px;`，元素的高度此时就会被拉伸到距离顶部和底部10px的位置。这时元素其实被赋予了一个适应于父级容器的自适应高度。像对于设置了`top: 0; bottom: 0; left: 0; right: 0;`的情况下，其实就是元素将被拉伸到填充满整个父级容器。
```css
.father-container {
  height: 300px;
  width: 70%;   
  background: #eee;
  margin: 10px auto;  /*父元素block的水平居中*/
  position: relative;
}

/*未居中的子元素*/
.child-box {
  top: 0;
  bottom: 0;
  left: 0;
  right: 0; 
  background: #222;
  position: absolute;
}
```
![absolute](/img/17-6/27-3.png)

我们可以看到`father-container`已经被`child-box`给完全填充掉了。而处于具有流体特性绝对定位元素的margin:auto的填充规则和普通流体元素一模一样：

1. 如果一侧定值，一侧auto，auto为剩余空间大小；
2. 如果两侧均是auto, 则平分剩余空间；

所以这个时候绝对定位元素才能通过这个方式实现完全居中。但是也由于使用了流体特性的原因，在没有设置的宽度和高度的情况下，子容器的宽度和高度也被会自适应的拉伸，因此就不会因为子元素的文本而被拉伸。所以我们需要手动的未元素设置宽度和高度。

使用此类居中，内部文本或者元素过多还会存在溢出的问题，我们可以通过`overflow: auto`来为其添加滚动条显示。







