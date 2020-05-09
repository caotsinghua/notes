 `defineReactive` 函数的核心就是 **将数据对象的数据属性转换为访问器属性**，即为数据对象的属性设置一对 `getter/setter`，但其中做了很多处理边界条件的工作。 

## dep

每个属性通过闭包使用自己的dep。

每个对象/对象属性有自己的dep。

```
const data = {
  a: {
    b: 1,
    __ob__: {value, dep, vmCount}
  },
  __ob__: {value, dep, vmCount}
}
```

一个对象被观测后会形成这样的结果。

 对于属性 `a` 来讲，访问器属性 `a` 的 `setter/getter` 通过闭包引用了一个 `Dep` 实例对象，即属性 `a` 用来收集依赖的“筐”。除此之外访问器属性 `a` 的 `setter/getter` 还通过闭包引用着 `childOb`，且 `childOb === data.a.__ob__` 所以 `childOb.dep === data.a.__ob__.dep`。也就是说 `childOb.dep.depend()` 这句话的执行说明除了要将依赖收集到属性 `a` 自己的“筐”里之外，还要将同样的依赖收集到 `data.a.__ob__.dep` 这里”筐“里，为什么要将同样的依赖分别收集到这两个不同的”筐“里呢？其实答案就在于这两个”筐“里收集的依赖的触发时机是不同的，即作用不同 。

 第一个”筐“里收集的依赖的触发时机是当属性值被修改时触发，即在 `set` 函数中触发：`dep.notify()`。而第二个”筐“里收集的依赖的触发时机是在使用 `$set` 或 `Vue.set` 给数据对象添加新属性时触发，我们知道由于 `js` 语言的限制，在没有 `Proxy` 之前 `Vue` 没办法拦截到给对象添加属性的操作。所以 `Vue` 才提供了 `$set` 和 `Vue.set` 等方法让我们有能力给对象添加新属性的同时触发依赖，那么触发依赖是怎么做到的呢？就是通过数据对象的 `__ob__` 属性做到的。因为 `__ob__.dep` 这个”筐“里收集了与 `dep` 这个”筐“同样的依赖。 



## 数组的观测

当定义了一个观测对象是这样的

```
data={
	arr:[
		[
		  {val:1}
		]
	]
}
```

1. 调用observe()方法观测data，给data加上`__ob__`。

2. 对data中每个属性，执行defineReactive,即对每一个属性都设置get/set进行依赖收集和更新等。因此对于arr这个属性，添加上get/set。

3. 对arr的值进行observe（如果不是对象类型的值是无效的），这里是数组，因此可以执行。

   对应这行：

   ```
   let childOb = !shallow && observe(val) // 对val执行observe，如果val是对象才有结果
   ```

   于是在第二步给data.arr加上dep后，data.arr的值也有了ob，因此给childOb加上依赖。

   由于值是一个数组，因此对数组中的每个元素，都要加上依赖（经过第三部的observe，childOb已经递归观测过，其中的每个元素都有ob）。需要注意的是，数组不会经过defineReactive添加依赖，而是在第二步中，由于子数组都观测过，因此通过dependArray添加依赖。

   

   对于数组，劫持其push，splice等方法，在进行这些操作时，首先对inserted的（即新添加的）部分进行observeArray，即对新添加的每个值都observe。然后，触发arr的dep的更新。（因为执行的变更操作）。

   

   数组操作拦截方法添加完毕后，对数组的元素递归observe，即对arr[0]执行observe，发现是一个数组，然后监测完毕后，又对`arr[0][0]`观测，是一个对象，然后触发walk。

   从而观测完毕。



// now：

1. global init 
2. prototype init
3. options merge
4. instance init
   1. data observed

