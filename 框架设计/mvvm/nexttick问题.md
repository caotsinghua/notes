## nexttick问题

### 现状

**v2.6**

对于nexttick的队列更新，使用microstask/macrotask的规则如下

1. 支持promise则用promise，`microTask`

2. 支持则用原生的MutationObserver,`microtask`

   通常在phantomjs，ios7，android4.4可用。

   在ie11不可靠。[#6466](https://github.com/vuejs/vue/issues/6466)

3. 支持则用原生setImmediate，`macro`

   虽然是macro，但是比settimeout优先级高。通常只有ie edge高版本可用。

4. 使用setTimeout，`macro`

**v3.0**

在3.0版本中，由于只支持ie11+，只用了promise。



### 问题

> 在2.6 nexttick部分提到的issue

1. [#6466](https://github.com/vuejs/vue/issues/6466)

   问题：mutationObserver在ie11有时会出现问题。表现为在设置v-model的input上同时按下3个按键，重复的值不合预期。

   解决：在ie下不使用mutationObserver

2. #[6813]( https://github.com/vuejs/vue/issues/6813 )

   问题：使用macrotask，触发的时机在repaint之后。当先更新state，后触发视图更新，则state引起的变化会在视图更新后。

   解决：由原来的先setimmediat，再messagechannel，promise，settimeout，即优先macrotask，改为分情况使用macrotask。对于@event.once的事件使用macro，其余都用micro。

3. macrotask的一些怪异问题

   - [#7109]( https://github.com/vuejs/vue/issues/7109 )

     In Vue 2.5 ，the use of MessageChannel in nextTick function will lead to audio can not play in some mobile browsers.

     messagechannel会引起一些问题。

     >  为何要使用MessageChanel来模拟task，而不是直接使用setTimeout
     > w3c html规范中定义了setTimeout的默认最小时间为4ms，而嵌套的timeout表现为10ms，也就是说即使你赋值0，而实际却不是。再由于，setTimeout的时间，会受到任务队列的影响（或其它原因？）其实际时间远大于10ms 

   - [#7153]( https://github.com/vuejs/vue/issues/7153 )

     问题：v-model输入数字时没有显示，第二次按下才显示。使用lazy无问题。

     修复： fix: should not update in-focus input value with lazy modifier 

     设置lazy时，focus的input不应触发更新。

   - [#7546]( https://github.com/vuejs/vue/issues/7546 )

     iPad/iPhone keyboard is not shown when input gets focus

     ios触发focus不弹出输入框的问题。

     >  iOS requires user input for showing keyboard. Before Vue uses micro task so user click counts as a valid trigger for keyboard. However, in 2.5 we uses macro task so iOS no longer treats it as trigger. 

   - [#7834 in 2.5.16]( https://github.com/vuejs/vue/issues/7834 )

     Unable to simulate click in `$nextTick`

      Modal not triggered in `$nextTick` but triggered by `setTimeout`. 

     可能还是MessageChannel引起的问题

   - [#8109 in 2.5.16]( https://github.com/vuejs/vue/issues/8109 )

     >  `requestAnimationFrame` has too long a delay and is quite indeterministic. We will likely revert to always using microTasks. Using `MessageChannel` fixed a few edge cases but caused more problems in the end... 
     >
     > 

4. microtask的问题

   - [4521 in 2.1.6]( https://github.com/vuejs/vue/issues/4521 )

      https://github.com/vuejs/vue/commit/36193183e124f0f1a054170d2cefd335d45135b8 

     ```
     // #4521: if a click event triggers update before the change event is
         // dispatched on a checkbox/radio input, the input's checked state will
         // be reset and fail to trigger another update.
     ```

     先触发了click，再触发了change，导致checkbox/radio重置

   - [6690 in 2.4.4]( https://github.com/vuejs/vue/issues/6690 )

     Select value is not updated correctly when input handler triggers class change

     select后，值改变，界面不变。

     >  FYI IE and Edge do not trigger `input` on `` elements, so you should probably use `change` instead anyway... 

     修复： [fix(v-model): update model correctly when input event is bound](https://github.com/chriscasola/vue/commit/bbd13afa0930cbc5b99e11d460c599a54c4ccc0f)  

   - [6566 in 2.4.2]( https://github.com/vuejs/vue/issues/6566 )

     ```html
     <div class="header" v-if="expand"> // block 1
       <i @click="expand = false, countA++">Expand is True</i> // element 1
     </div>
     <div class="expand" v-if="!expand" @click="expand = true, countB++"> // block 2
       <i>Expand is False</i> // element 2
     </div>
     ```

     > So, this happens because:
     >
     > - The inner click event on `` fires, triggering a 1st update on nextTick (microtask)
     > - **The microtask is processed before the event bubbles to the outer div**. During the update, a click listener is added to the outer div.
     > - Because the DOM structure is the same, both the outer div and the inner element are reused.
     > - The event finally reaches outer div, triggers the listener added by the 1st update, in turn triggering a 2nd update.
     >
     > This is quite tricky in fix, and other libs that leverages microtask for update queueing also have this problem (e.g. Preact). React doesn't seem to have this problem because they use a synthetic event system (probably due to edge cases like this).
     >
     > To work around it, you can simply give the two outer divs different keys to force them to be replaced during updates. This would prevent the bubbled event to be picked up:
     >
     > ```
     > <div class="header" v-if="expand" key="1"> // block 1
     >   <i @click="expand = false, countA++">Expand is True</i> // element 1
     > </div>
     > <div class="expand" v-if="!expand" @click="expand = true, countB++" key="2"> // block 2
     >   <i>Expand is False</i> // element 2
     > </div>
     > ```

