需要处理的$options

1. el,propsData

   生产环境：defaultStrat，即child存在就child，否则parent

2. data

   区分有vm和没有vm的情况。

   - 有vm- 说明是new 实例

     data会返回mergedInstanceDataFn的方法，在该方法中，返回mergeData

     > *以child的值为准，如果child没有，则拿parent的，如果child有，则使用child的，（如果两个是对象，则对这个值再进行上述操作进行深度合并）*

   - 没有vm，说明是Vue.extend

     创建一个子类，并且data必须是函数。

     child和parent如果一个不存在，则返回另一个。

     如果都存在&都为函数，则对函数执行结果进行mergeData。返回mergedDataFn。

3. hooks

   包括

   ```
   // [
   //   'beforeCreate',
   //   'created',
   //   'beforeMount',
   //   'mounted',
   //   'beforeUpdate',
   //   'updated',
   //   'beforeDestroy',
   //   'destroyed',
   //   'activated',
   //   'deactivated',
   //   'errorCaptured',
   //   'serverPrefetch'
   // ]
   ```

   ```
   // parentVal 一定是数组吗？答案是：如果有 parentVal 那么其一定是数组，如果没有 parentVal 那么 strats[hooks] 函数根本不会执行。因为只有parent中存在key时，才会进行mergeField
     // if(childVal){
     //   if(parentVal){
     //     res=parentVal.concat(childVal);
     //   }else{
     //     if(Array.isArray(childVal)){
     //       res=childVal
     //     }else{
     //       res=[childVal]
     //     }
     //   }
     // }else{
     //   res=parentVal
     // }
   
   // 最终按照parent.concayt(child)的顺序来执行某个钩子
   ```

   对于hooks，最终一定返回一个去重后的数组。child在parent顺序后。

   当parent不存在钩子&子元素不存在钩子，mergeField不会执行。

   parent不存在钩子，子元素存在。即从根Vue开始操作时会发生。此后的子类，其父元素一定有钩子了，并且时数组。

   如 Vue.extend({create(){}}),之后，该实例的钩子就是数组了。

4. 资源

   包括：*[component,filter,directive]*

   如果父options有资源，如components，

   那么：`const res = Object.create(parentVal || null) *// 如果parent有资源，就作为__proto__添加到res*`

   这之后如果子options也有components，就extend到对象本身的属性中去。

   > 在initGlobalApi时，Vue.options的资源已经初始化过
   >
   > ```
   >  extend(Vue.options.components, builtInComponents)
   > ```
   >
   > 

5. watch

   >  nativeWatch
   >
   > 在 `Firefox` 中原生提供了 `Object.prototype.watch` 函数，所以当运行在 `Firefox` 中时 `nativeWatch` 为原生提供的函数，在其他浏览器中 `nativeWatch` 为 `undefined`。这个变量主要用于 `Vue` 处理 `watch` 选项时与其冲突。 

 最终watch会返回一个对象，他的value是一个数组（当parent有watch时），也可能是普通函数或对象（当直接返回childVal时）。

```js
strats.watch = function (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string // key是watch
): ?Object {
  // work around Firefox's Object.prototype.watch...
  // 在 Firefox 中原生提供了 Object.prototype.watch 函数，所以当运行在 Firefox 中时 nativeWatch 为原生提供的函数，在其他浏览器中 nativeWatch 为 undefined。这个变量主要用于 Vue 处理 watch 选项时与其冲突。
  if (parentVal === nativeWatch) parentVal = undefined
  if (childVal === nativeWatch) childVal = undefined
  /* istanbul ignore if */
  if (!childVal) return Object.create(parentVal || null) // 如果child options.watch不存在，那么用parentoptions.watch作为原型创建对象并且直接返回
  if (process.env.NODE_ENV !== 'production') {
    assertObjectType(key, childVal, vm)
  }
  if (!parentVal) return childVal // 如果prent options .watch不存在，就直接返回child的watch
  const ret = {}
  extend(ret, parentVal) // 拷贝parent options。watch
  for (const key in childVal) {
    let parent = ret[key]
    const child = childVal[key]
    // 获取child watch的key在两个对象中的值
    if (parent && !Array.isArray(parent)) {
      parent = [parent]
    }
    // parent最终成为对象
    ret[key] = parent
      ? parent.concat(child)
      : Array.isArray(child) ? child : [child] // 结果是parent.concact的数组
  }
  // ret是一个对象，值为数组
  return ret
}
```

6. props,methods,inject,computed 

   都是纯对象，用child覆盖parent的值

7. provide 

   provide的参数也是一个对象/方法，因此和data一样，返回mergeDataOrFn.

   

