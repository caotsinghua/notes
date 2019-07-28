**登陆**

如果使用`/user/status`的方式获取登陆状态，逻辑为

```flow
st=>start: Start
op=>operation: 进入页面（任意界面）
cond=>condition: getStatus
saveToStore=>operation: 保存登陆状态到store
isLogged=>condition: store.user.isLogged
routerBeforeEach=>operation: beforeach检查登陆态
next=>operation: next 下一个路由
nextFalse=>operation: 如果去的界面不是登陆界面给出提示并跳到登陆
end=>end

st->op->cond
cond(yes)->saveToStore->routerBeforeEach->isLogged
cond(no)->op
isLogged(yes)->next->end
isLogged(no)->nextFalse
```



>  如果在`App.vue`即存放根router-view的文件中，在mounted时`getStatus`，此时路由同时在进行跳转，即进行beforeEach的判断，而`beforeEach`时vue实例还没有加载，直接访问store获取不到store.user.isLogged状态。



- 方案1：在main.js中vue实例未创建前就获取getStatus

  由于getStatus在进入应用时只需调用一次，因此在main.js中调用也未尝不可，并且在beforeEach前store已经设置好登陆态。

  问题在于如果`getStatus`接口延时太长或接口错误，会导致`vue`实例不创建而长时间白屏。

  可以直接在`public/index.html`设加载界面代替白屏。

  ```javascript
  async function main() {
      try {
          await store.dispatch('getUserInfo');
      } finally {
          new Vue({
              el: '#app',
              router,
              i18n,
              store,
              render: h => h(App)
          });
      }
  }
  ```

一些问题：

- 可以确定的是，使用该方式在确定登陆状态前应用是不知道跳转哪个界面的。

- 如何监听到store的状态映射到if(store.xxx===xxx)？以便延迟beforeEach中的逻辑执行