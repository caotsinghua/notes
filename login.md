## 登录

### getStatus判断登录态

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



>  如果在`App.vue`文件中，在mounted时`getStatus`，此时路由同时在进行跳转，即进行beforeEach的判断，而`beforeEach`时请求getStatus也在同时进行，直接访问store是获取不到store.user.isLogged状态的。



- 方案：在main.js中vue实例未创建前就获取getStatus

  由于getStatus在进入应用时只需调用一次，因此在main.js中调用也未尝不可，并且在beforeEach前store已经设置好登陆态。

  问题在于如果`getStatus`接口延时太长或接口错误，会导致`vue`实例不创建而长时间白屏。

  ~~可以直接在`public/index.html`设加载界面代替白屏。~~

  ***可以在main.js设置加载状态代替白屏。***
  
  ```javascript
  const main = async () => {
      iView.Spin.show();
      try {
          await store.dispatch('getUserStatus');
      } catch (e) {
          iView.Message.error(e.message || '获取登录态错误');
      } finally {
          /* eslint-disable no-new */
          new Vue({
              el: '#app',
              router,
              i18n,
              store,
              render: h => h(App)
          });
          iView.Spin.hide();
      }
  };
  ```

一些问题：

- 可以确定的是，使用该方式在确定登陆状态前应用是不知道跳转哪个界面的。
- ~~如何监听到store的状态映射到if(store.xxx===xxx)？以便延迟beforeEach中的逻辑执行~~

### iview-admin登录逻辑-getUserInfo+token

```flow
start=>start: 开始
end=>end: 结束
tiaozhuan=>operation: 跳转界面
beforeEach=>operation: beforeEach
getToken=>operation: 取token
setToken=>operation: 存token
getInfo=>operation: 获取用户信息(失败则跳转登录)
hasToken=>condition: 是否有token
judgeLogin=>condition: 是否登录态
judgeGetinfo=>condition: 是否获取过用户信息（初次进入或刷新时store里一定是false）
toPage=>condition: 判断权限决定跳转到的页面
start->tiaozhuan->beforeEach->getToken->hasToken
hasToken(yes)->judgeGetinfo
hasToken(no)->tiaozhuan
judgeGetinfo(yes)->toPage->end
judgeGetinfo(no)->getInfo->toPage->end
```

需要注意的地方：

1. 有token&去登录页->跳转home->4（还要进入beforeEach判断）

2. 没有token&去登录页->next

3. 无token&去非登录页->去登录页->2

4. 判断store.hasGetInfo,false则调getInfo接口查询用户信息->获取成功说明用户在登录态->5；获取失败说明用户在服务端未登录->清除token去登录页->2
5. 判断用户权限决定跳转的界面(store已设置好hasGetInfo)，有权限则进入指定页面

综上：使用token和getInfo判断登录态在逻辑上和getStatus方式基本一致

**比较**

- 首次进入/刷新 都会调用getInfo/status来检查服务端用户登录信息
- 当本地无token而服务端处于登录态时：getUserInfo的方式在进入系统时会跳转登录页重新登录；而getStatus的方式由于登录态完全由服务端决定所以会跳转首页。

- 在本地无token情况下，getUserInfo方式直接跳首页；而getStatus方式会调接口从而存在白屏
- 在本地有token情况下，getUserInfo也会判断登录态，也存在白屏

