### **前言**

可能很多同学都见过 NodeList 和 HTMLCollection，但是未必能讲出个所以然。而这俩家伙就是动态集合，另外还有一个少见的 NamedNodeMap，加在一起这仨兄弟就是我要讲的 Javascript 中的动态集合。

开始前先介绍下我们最熟悉的 DOM。DOM 是 javascript 操作网页的接口，全称为文档对象模型(Document Object Model)。它的作用是将网页转为一个 javascript 对象，从而可以使用 javascript 对网页进行各种操作(比如增删内容)。浏览器会根据 DOM 模型，将 HTML 文档解析成一系列的节点，再由这些节点组成一个树状结构。DOM 的最小组成单位叫做节点(node)，文档的树形结构(DOM 树)由 12 种类型的节点组成。参见：[深入理解 DOM 节点类型第一篇——12 种 DOM 节点类型概述](https://www.cnblogs.com/xiaohuochai/p/5785189.html)

而动态集合其实就是节点的集合，其中动态则是指 DOM 结构的变化能够自动反映到所保存的对象中。这仨兄弟且都是类数组对象。

> 未特殊声明，以下所有 demo 代码都用这段 html 结构：

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Javascript动态集合</title>
</head>
<body>
  <div id="divs">
    <!-- 这里是注释 -->
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
    <div class="item">4</div>
    <div class="item">5</div>
  </div>
</body>
</html>
```

### **NodeList**

NodeList 是 node 节点(包含 node 节点中的 12 种节点)的集合，保存了一组有序的节点，是由 `Node.childNodes` 和 `document.querySelectorAll` 返回的一个类数组对象，前者是动态的，后者是静态的。这是 NodeList 跟另外俩兄弟的最大不同之处。

- 首先看 childNodes 属性中的 NodeList 对象：

```
let divs = document.getElementById('divs')
let items = divs.childNodes
console.log(items, items.length)
```

打印结果：
![childNodes_NodeList](https://github.com/Marze1994/blog/blob/master/images/js-dynamic-collection/childNodes_NodeList.jpg)

> 结果中的 0、2、4、6、8、10、12 为文本节点；1 为注释节点；3、5、7、9、11 为元素节点。

- 再来看看 document.querySelectAll()方法返回值中的 NodeList 对象：

```
let divs = document.querySelectorAll('div')
console.log(divs, divs.length)
```

打印结果：
![querySelectorAll_NodeList](https://github.com/Marze1994/blog/blob/master/images/js-dynamic-collection/querySelectorAll_NodeList.jpg)

> 此时结果都为元素节点。

那为啥说 Node.childNodes 是动态集合而 document.querySelectorAll 是静态集合呢？看下图你会恍然大悟：
![dynamicVSstatic](https://github.com/Marze1994/blog/blob/master/images/js-dynamic-collection/dynamicVSstatic.jpg)

### **HTMLCollection**

HTMLCollection 和 NodeList 类似，但是不同之处在于，NodeList 集合包含了 node 节点中 12 种节点，而 HTMLCollection 仅包含 Element 元素节点的集合。

HTMLCollection 集合包括 `getElementsByTagName()`、`getElementsByClassName()`、`getElementsByName()` 等方法的返回值，以及 `children`、`document.links`、`document.forms`、`document.anchors`、`document.images` 等元素集合。比如：

```
let divs = document.getElementsByTagName('div')
console.log(divs, divs.length)
```

打印结果：
![getElementsByTagName_HTMLCollection](https://github.com/Marze1994/blog/blob/master/images/js-dynamic-collection/getElementsByTagName_HTMLCollection.jpg)

### **NameNodeMap**

可能有些同学没有听过 NamedNodeMap 对象。我们都知道 DOM 中的 Element 元素节点是唯一拥有 `attributes` 属性的一种节点类型，attributes 属性返回的集合即为 NamedNodeMap，例如 id、class、title 等。

### **注意事项**

由于他们仨兄弟都是类数组对象，所以都具有 length 属性，此属性也是用的最多的，至于其他不常用的方法属性可以参见 [NodeList | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/NodeList)、[HTMLCollection | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCollection)、[NamedNodeMap | MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/NamedNodeMap)
虽然都为动态集合，但有一点一定要注意，在使用循环时有可能会造成死循环。例如：

```
let divs = document.getElementsByTagName('div');
for(let i = 0 ; i < divs.length; i++){
    document.body.appendChild(document.createElement('div'));
}
```

上面代码中，由于 divs 是一个 HTMLElement 动态集合，divs.length 会随着 appendChild()方法，而一直增加，于是变成一个死循环。
为了避免此情况，一般可以写成下面形式：

```
let divs = document.getElementsByTagName('div');
for(let i = 0,len = divs.length; i < len; i++){
    document.body.appendChild(document.createElement('div'));
}
```
