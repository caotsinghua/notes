1. 如何实现数据响应

   - Object.defineProperty (ie9+)

     todo

   - Proxy

     todo

2. 如何避免重复依赖收集

   参考第五点中部分内容。

   deps: 上一次收集的依赖

   newDeps:这次get收集的依赖。每次一开始都是空的。

   - 在addDep时检查newDep是否存在这个dep，如果不存在才加入newDep如果deps中没有这个值，即上一次也没有这个依赖，就addSub 

   - get结束后，检查deps中有没有newDeps中不存在的依赖，不存在表示这个值不再监听，从dep.subs中移除这个watcher。然后 设置deps是newDeps并清空newDeps。这样就保存上一次收集依赖。

     

3. 如何实现深度监测

   watcher进行get的时候，对于值为对象数组类型的，对其key进行遍历取值即可。

   避免循环引用，可以通过`value.__ob__,dep._uid`检查是否取值过。

4. 数据变化后，如何保证新的值后续变更也能触发callback？

   1. set检查到赋值是一个对象类型，则进行深度observe并赋值给childOb，下次依赖收集时，对子值的依赖收集就使用新的childOb。

   2. watcher.update时，利用watcher.get重新触发依赖收集。

   需要注意的是，这里可能会重复收集依赖。

   

5. 为什么数据get的时候调用dep.depend=>target:watcher.addDep=>dep.addSub(target:watcher) ?

   为什么不直接dep.addSub(target)呢？

   因为直接addSub会重复收集依赖。如何确定在一次getter时只给值的dep唯一的watcher？ 

   每个reactive的值都有一个对应的dep。可以认为一个dep对应一个值。如果在dep加入watcher时该watcher已经存在说明已经收集过。

   需要达成的效果：

   - 一次watcher.get时，被用到的值只有唯一的dep被收集依赖。

   - 多次get时（后续更新可能会有一些值不再需要watch），需要去除不需要的值的依赖，并且与前一次不重复，即一个值两次都用到，第二次不收集。

   也就是说，watcher不能重复添加一样的dep，这样，这个dep也就不会添加重复的watcher 了。

   watcher和dep的关系：

   - watcher里有多个使用到的值的dep。

   - dep有多个监听自己值的watcher。

   ```
   
   watcher1 => { a,a,b,c }
   这个watcher监听了a,b,c三个值，其中a用了两次。
   
   目标
   watcher1.deps=[a,b,c]
   a.subs=[wacther1] // 重复是[watcher1,watcher1]
   b.subs=[watcher1]
   c.subs=[watcher1]
   
   所以要完成
   添加watcher到dep,添加dep到watcher的操作。
   其实在dep/watcher中进行都一样。都要拿到target和当前dep进行操作。
   最终要避免重复。
   
   
   ```

6. 如何实现computed？

   假设使用规范化定义：

   ```
   computed:{
   	num:{
   		get(){ return this.money},
   		set(val){ this.money=val }
   	}
   }
   ```

   不讨论set。

   1.把get作为getter给watcher进行依赖收集，不设回调。设lazy=true触发惰性求值。

   这样，get里用到的值发生更新了并不会触发什么变化。

   2.如何在用到的值变化后自动更新computed值？ 劫持num的get，将num的值改为从watcher.value获取，每次获取值时，都会进行evaluate获取最新的值。

   3.如何触发视图更新？ 当渲染函数使用到num，则num的dep会加入渲染函数的watcher，当num变化就触发更新。但是computed属性是没有被observe的，从哪里得到dep呢？当watch computed值的时候，会将get里的dep都加入到watcher中，因此实际上渲染函数的watcher是添加到get中的值的dep中的。实际上是当用到的值变化更新->视图更新，而不是说num发生变化才更新，num的值只在触发其劫持后的get才更新。

7. computed缓存？

   当获取computed的值的时候，执行watcher.evaluate求值。

   当dirty=true时重新触发watcher.get,再将dirty=false。

   那么什么时候dirty=true?在依赖的值发生更新的时候，将dirty=true，这样下次获取computed值的时候一定是最新的。并且获取完，dirty又变为false，意味着如果值没有更新，那么computed获取值时永远是之前的那个值，不会重新计算（即使重新计算value也不会变）

### ps:

1. 数据observe了只是加上一个劫持，具体触发变化还需要绑定watcher到dep。没有watcher也是不会触发更新操作的。

2. 每个被defineReacttive的值都有一个dep，默认是空的。

   当watch的getter中有这个值时，在watch时（new Watcher）用getter进行依赖收集并赋值给watcher。（需要注意，虽然触发值的get拦截器会进行依赖收集，但不经过watcher触发的话就没有Dep.target，收集依赖是无效的）。

   这样当这个值set时，就会将dep中的watcher全部update（即所有监听了这个值的watcher）。

   watcher执行update时，会再次用getter收集依赖并获取新的值。随后执行回调。

   这样一轮依赖收集+更新的流程就完成了。

