## 重排reflow与重绘repaint

### 重排（回流） reflow

reflow是指重新计算页面布局

某个节点reflow时会重新计算节点的尺寸和位置，而且还有可能触发其他子节点、祖先节点和页面上的其他节点reflow，并且在这之后再触发一次repaint。

当render tree中的一部分（或全部）因为元素的规模尺寸、布局、隐藏等改变而需要重新构建。这就称为回流，每个页面至少需要一次回流，就是再页面第一次加载的时候。


导致reflow的操作

- 调整窗口大小
- 改变字体
- 增加或移除样式表
- 内容变化，比如用户在input框中输入内容
- 激活css伪类，比如 ：hover
- 操作class属性
- 脚本操作DOM
- 计算offsetWidth 和 offsetHeight 属性
- 设置style属性的值


触发页面重布局的一些css属性

- 盒子模型相关属性会触发重布局
  - `width`
  - `height`
  - `padding`
  - `margin`
  - `display`
  - `border-width`
  - `border`
  - `min-height`

- 定位属性及浮动也会触发重布局
  - `top`
  - `bottom`
  - `left`
  - `right`
  - `position`
  - `float`
  - `clear`
 
- 改变节点内部文字结构也会触发重布局
  - `text-align`
  - `overflow-y`
  - `font-weight`
  - `overflow`
  - `font-family`
  - `line-height`
  - `vertacal-align`
  - `white-space`
  - `font-size`
  
  
### 重绘 repaint

repaint或者redraw遍历所有节点检测各个节点的可见性, 颜色, 轮廓等可见样式属性, 然后根据检测出来的结果更新页面的响应部分

当render tree中一个元素需要更新属性, 而这些属性只是影响元素的外观, 风格, 而不会影响布局的, 比如background-color, 这就叫做重绘

只触发重绘不触发回流的一些css属性
- `color`
- `border-style`/`border-radius`
- `visibility`
- `background`
- `outline`
- `box-shadow`

我们来看一个例子：

```javascript
var bstyle = document.body.style; // cache

bstyle.padding = "20px"; // reflow, repaint
bstyle.border = "10px solid red"; //  再一次的 reflow 和 repaint

bstyle.color = "blue"; // repaint
bstyle.backgroundColor = "#fad"; // repaint

bstyle.fontSize = "2em"; // reflow, repaint

// new DOM element - reflow, repaint
document.body.appendChild(document.createTextNode('dude!'));
```

我们的浏览器是聪明的,他不会像上面那样, 你每一次该样式, 它就reflow或者repaint一次.

一般来说浏览器会把这样的操作攒一攒, 然后做一次reflow, 这又叫异步reflow或者增量异步reflow.

但是有些情况浏览器是不会这么做的, 比如: resize窗口, 改变页面默认的字体等. 对于这些操作, 浏览器是会立即惊醒reflow




**减少重绘与重排：**

重绘和回流在实际开发中是很难避免的, 我们能做的就是尽量减少这种行为的发生

- js尽量少访问dom节点和css属性, 尽量不要过多的频繁的去操作, 这可能会频繁的导致页面reflow, 可以先把改dom节点抽离到内存中进行复杂的操作, 然后再display到页面上(fragment 内存文档碎片)
  
- 减少不必要的dom层级. 因为改变dom树种一级会导致所有层级的改变, 上至根部, 下至被改变节点的子节点, 这导致大量的时间耗费在reflow上面

- 不要通过父级来改变子元素样式，最好直接改变子元素样式，改变子元素样式尽可能不要影响父元素和兄弟元素的大小和尺寸

- 尽量通过class来设计元素样式，切忌用style 多次操作单个属性

  ```javascript
  // bad
  var left = 10,
  top = 10;
  el.style.left = left + "px";
  el.style.top  = top  + "px";
  
  // Good
  el.className += " theclassname";
  
  // Good
  el.style.cssText += "; left: " + left + "px; top: " + top + "px;";
  ```
 
- 尽可能为产生动画的html元素使用`fixed` 或 `absolute`的`position`, 那么修改他们的css是不会reflow的

- img标签要设置宽高, 以减少回流和重绘

- 把dom脱离文档流后, 如果先 `display: none` , 再修改属性, 这样就只会发生一次回流

- 尽量用 `transform` 来做形变和位移，不会造成回流

- 权衡速度的平滑。比如实现一个动画，以1个像素为单位移动这样最平滑，但reflow就会过于频繁，CPU很快就会被完全占用。如果以3个像素为单位移动就会好很多。

- 不要用tables布局的另一个原因就是tables中某个元素一旦触发reflow就会导致table里所有的其它元素reflow。在适合用table的场合，可以设置table-layout为auto或fixed，

- 避免不必要的复杂的 CSS 选择器，尤其是后代选择器（descendant selectors），因为为了匹配选择器将耗费更多的 CPU。

> PS:  display:none会触发reflow，而visibility:hidden只会触发repaint，因为没有发现位置变化


#### 总结

重排(回流): 元素的尺寸变了, 位置变了

重绘: 元素的颜色, 边框, 轮廓变了, 但是元素的几何尺寸没有改变

reflow的成本比 repaint的成本高的多, DOM tree里面的每一个节点都会有reflow方法, 一个节点的reflow很有可能导致子节点, 甚至父节点以及同节点的reflow.

在一些高性能的电脑上也许没有什么问题, 但是reflow发生在手机上, 那么这个过程是非常痛苦和耗电的









































