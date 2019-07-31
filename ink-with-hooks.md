```javascript
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

> 错误示例

```javascript
useEffect(() => {
    if (setRawMode) setRawMode(true);
    stdin.on('data', handleInput);
    return () => {
      stdin.removeListener('data', handleInput);
      if (setRawMode) setRawMode(false);
    };
  }, []);

const handleInput = useCallback(
    async (data: any) => {
      const s = String(data);
      if (s === CTRL_C) process.exit(0);
      switch (state.view) {
        case VIEW.SEARCH: {
          // if (s === ARROW_DOWN || s === ENTER || s === SPACE) {
          //   setView(VIEW.SCROLL); // 设滚动窗口为active
          // }
          dispatch(setView(VIEW.SCROLL)); // 设滚动窗口为active
          break;
        }
    ............
```

如上代码所示，错误原因在于useEffect只在第一次加载时绑定事件，而绑定的handleInput函数是一个闭包，里面的state永远是第一次初始化的值。

解决方案：

- 取消[]
- 