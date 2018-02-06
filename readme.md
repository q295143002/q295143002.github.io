Online CSS 架构与实现方案
===
# ![](https://pic.english.c-ctrip.com/picaresenglish/ibu/ibucommon/images/v1/trip-logo.e7bdd3cf.png)

##### Powered by [Sem小组](ibu_sem@Ctrip.com)

###### Created by Neo xia ( [@Neo](xia_g@ctrip.com) )

---

# 目录
- **业界一般文件结构做法分析**
- **目前Online CSS分析**
- **H5与online CSS可复用性分析**
- **CSS命名规范以及设计规范**
- **Online CSS文件目录结构**


---
## 业界一般文件结构做法分析
- `header.css`, `body.css`, `footer.css`
- `reset.css`, `main.css`, `content.css`
- `common.css`,然后每个页面一个单独CSS，如：`home.css`,`order.css`
- 直接的内联样式

---


****当然出了以上的文件结构还有其他的文件结构，这里不再列举，这些结构针对其业务类型等原因都有其存在的道理，并没有谁好谁坏，这里的我们的css文件大方向上采用第二种的文件结构****

---

## 目前Online CSS分析
- 结构分析
 目前online CSS文件分为两块，一块为我们自己维护的`hotels.css`,
 一块为公共维护的`global.css`。并未加载内联样式，截图如下：
> ![](pic.png)
>整体结构并没有什么大问题，在接下来的结构调整中，我们会抽离出一部分的首屏加载CSS作为内联样式，其余的依然按照原有的结构。

---

- 大小与冗余度分析
如结构分析截图所示`global.css`和`hotels.css`大小分别为`41.3kb`,`40.9kb`,但是使用率如何呢？我们看一下截图：
> ![](pic1.png)
>明显可以看出两个CSS的使用率只有10%左右，考虑到响应式的其他css未被考虑到，使用率最多在20%左右。在接下来的重构过程中，我们会把`global.css`里的冗余样式清除合并到我们自己的CSS中，弊端就是<font color='red'>会不能及时更新公共的样式</font>，需不需要这样做，需要讨论。基于目前online的css大小，我们的css Gzip大小应该小于8K。


## H5与online CSS可复用性分析
- **H5概括**
  H5的整体架构是以Rem为基础单位设计，兼容性为现代浏览器，不支持IE，小图标使用的是fonts文件，支持类似于`css 高级语法`；
- **Online概括**
  online的整体架构是以px为基础单位设计，兼容性为ie7+,所以小图标用sprite picture，并不完全支持`css 高级语法`，对于某些效果在低版本的浏览器中我们可能会适当舍弃。
---
- **H5 Online可复用组件分析**
  如果要在Online和H5上进行组件样式复用，有以下几点需要考虑
  1. 结构需要一致
  2. 组件拆分要颗粒化
  3. 样式的类名不要有父级
  >以手风琴组件为例，结构如下：
 ```
<div class='container'><!--页面的类，与组件无关-->
    <div class="panel panel-success">
      <div class="panel_header"></div>
      <div class="panel_body"></div>
    </div>
 </div>
```
>css如果做成如下模式就有问题：
```
.container .panel{font-size: 12px}
.container .panel_header{font-size: 16px}
.container .panel_body{font-size: 156px}
```
>如果这样做的结果就是我在调用手风琴组件的时候，必须要给其父节点加上`.container`，这样的做法是有问题的，所以故而在编写组件的时候，组件的根类一定是要独立的，以上结构一定要做成如下：
```
.panel{font-size: 12px}
.panel_header{font-size: 16px}
.panel_body{font-size: 156px}
```
***基于此，H5和Online可以做到服用的组件，有手风琴组件、酒店卡片组件（这个可以拆分成更小的，比如`badge`,`currency`等）***

---

- **其他可复用文件分析**
针对不涉及到业务或兼容性的Css，我们可以抽离出来做成一套，比如`reset.css`,`function.css`,`variable.css`,`util.css`等
---
## CSS命名规范以及设计规范
**采用`BEM`+`命名空间`**
BEM：block+Element+Modification
先上段代码
 ```
<div class='container'><!--页面的类，与组件无关-->
    <div class="panel panel-success">
      <div class="panel_header panel_header-active"></div>
      <div class="panel_body   panel_header-active">
      	  <ul class='panel_list'>
            	<li class='panel_item'></li>
            </ul>

      </div>
    </div>
 </div>
```
less的书写为如下模式：
```
.panel{/*styles*/}
.panel-success{color:green;}
.panel_header{}
.panel_header-active{}
```
你不必写如下less
```
.panel{
    .panel-success{color:green;}
    .panel_header{
    	panel_header-active{}
    }
}

```
如上代码编译将是一大串
如上代码Block 的class为`.panel`
Elemnet的命名规则为`.panel_**`
Modification 的命名规则为`.panel-**`
Elemnet的Modification的命明规则为`.panel_**-**`
永远别出现二层链接：
 ```
 <ul class='panel_list'>
    <li class='panel_item'>
    	<a class='panel_item_link'>hahaha</a>
    </li>
 </ul>
```
如上出现了`.panel_item_link`,这是不允许的，一般的解决方法如下：
```
<ul class='panel_list'>
    <li class='panel_item'>
    	<a class='item_link'>hahaha</a>
    </li>
 </ul>
```
去建立一个伪块`.item`,继续使用BEM命名方式；
而在less中是这样的：
```
.panel_item{
	.item_link{
      /*styles*/
    }
}

```
好处是：
1. 结构扁平,基本所有的Class优先级就在10
2. 你可以知道哪些元素是归属于一个block


命名空间：


























