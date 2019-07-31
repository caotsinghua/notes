```
const [value,setValue]=useState('')
handleChange(v){
setValue(v);
}

// 子组件
handleChange(parentValue+value);
//获取的parentValue永远是空
```

即当state更新后，子元素获取到的state值还是最初始的值。

期间debug发现子组件接受到了改变后的值，但是再一次输入后，state还是为空值。

同样的逻辑在web有效，在ink无效。

可以用useRef解决。

使用useReducer共享状态，子元素拿到的state值也还是初始值，无效。