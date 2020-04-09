## patchClass

对element的class直接更新。

针对svg使用setAttribute

其他元素：

如果有transitionClasses，则组合transitionClasses



## patchStyle

对于样式

1. 如果移除样式，则removeAttribute

2. 如果next是字符串，则直接替换

   style.cssText=next

   TODO：作用

3. 如果next是对象

   首先对原style添加属性

   > 判断自定义参数，import替换，添加前缀等

   然后对旧style（非字符串）遍历，删除无效属性。

## 事件

对于以on开头的属性

对于onUpdate以外的事件，patchEvent

其中，对于配置的对象不一样的

如capture，passive，once，先删除前一个监听，在添加。

如果有新监听函数，则添加到该事件。否则，取消前一个invoker的事件监听。

## 其他DOM Properties

更新dom标准属性

非svg元素，并且element存在这个prop属性

则执行patchDomProp

## 其他自定义attribute

针对checkbox的true-value和false-value特殊处理

其他的使用patchAttr

