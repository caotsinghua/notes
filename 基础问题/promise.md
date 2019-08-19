## promise

### 1.错误处理 

>**catch()** 方法返回一个[Promise](https://developer.mozilla.org/zh-CN/docs/Web/API/Promise)，并且处理拒绝的情况。它的行为与调用[`Promise.prototype.then(undefined, onRejected)`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 相同。 (事实上, calling `obj.catch(onRejected)` 内部calls `obj.then(undefined, onRejected)`).

> 1. 使用promise.catch可以捕捉到promise进行过程中的错误，一般是reject触发的。
>
> 2. 如果在promise.then过程中throw错误，可以在该promise的catch中再次获取到

```javascript
function promiseWithError_1(hasError=false){
            return new Promise((resolve,reject)=>{
                if(hasError){
                    reject("reject了一个错误");
                }else{
                    resolve("完成")
                }
            })
        }
        promiseWithError_1(true).then(res=>{
            console.log(res);
        }).catch(e=>{
            console.log(e);
        })
```

对上述代码来说，运行结果显然是reject的错误。

```javascript
function promiseWithError_2(hasError=false){
    throw Error("未返回promise之前出错了");
    return Promise((resolve,reject)=>{
        resolve("完成")
    })
}
// 调用1
promiseWithError_2().then(res=>{
    console.log(res);
}).catch(e=>{
    console.log(e);
})
// 调用2
try{
    promiseWithError_2().then(res=>{
        console.log(res);
    }).catch(e=>{
        console.log(e);
    })
}catch(err){
    console.log(err)
}
```

显然，调用1的catch是获取不到函数同步任务时的错误的，只有用try catch去捕捉函数执行过程中同步任务的错误，promise.catch只能捕捉当前promise执行过程中产生的错误

> 如果promise中不reject错误，而是直接throw 呢？

> _如果在executor函数中抛出一个错误，那么该promise 状态为rejected。executor函数的返回值被忽略。_

```javascript
function promiseWithError_3(){
    return new Promise((resolve,reject)=>{
        throw Error("在promise执行过程中出错了")
        resolve("asda");
    })
}
promiseWithError_3().then(res=>{
    console.log(res);
}).catch(e=>{
    console.log(e);

})
```

结果与reject一致。

**当promise的内层出错时，外面一层的promise的catch能不能处理呢？**

```javascript
// 内层promise抛出错误，外层能否处理？
function promiseInner(){
    return new Promise((resolve,reject)=>{
        reject("内层抛出错误")
        // throw Error("内层抛出错误")
    })
}
function promiseOut(str){
    return new Promise((resolve,reject)=>{
        console.log("[promiseOut] 执行了");
        resolve(str);
    })
}
function testOI(){
    promiseOut("outtt").then(res=>{
        console.log("res==")
        console.log(res);
        promiseInner().then(res2=>{
            console.log(res2);
        }).catch(e=>{
            console.log(e)
            throw("eee") //内部catch后再抛出，外层能否处理？
        })
    }).catch(e=>{
        console.log("在外层捕捉了错误",e)
    })
}
testOI()
```

答案是不能

- `promiseInner`的错误只能由它自己来处理，因此如果promise中可能存在错误，我们需要在每一个then后面都有catch去捕捉到。并且当第一次catch到后（或设置then的第二个参数），后面的catch就触发不到。
- 如果在`promiseInner`的catch中捕捉到错误，再抛出呢？该错误也只能promiseInner在接下来的catch中捕捉到，外层是取不到的

为什么promise捕捉不到内部promise的错误呢？

因为promise捕捉不到里面异步函数抛出的错误，在里面promise抛出错误和settimeout抛出错误是一样的，~~已经在当前microtask的后面？~~ 因为外层的promise已经resolve了

*resolve后抛出错误会被忽略*

```javascript
function throwAfterResolve(){
    return new Promise((resolve)=>{
        resolve();
        throw Error("error")
    })
}
throwAfterResolve().catch(e=>{
    console.log(e) // 不会执行
})
```

*但resolve后的代码将正常执行*

```javascript
function runAfterResolve(){
    return new Promise((resolve)=>{
        resolve();
        console.log("又执行了")
    })
}
runAfterResolve().catch(e=>{
    console.log(e)
})
```

promise中如果异步抛出错误是不会被当前catch捕获到的

```javascript
new Promise((resolve,reject)=>{
    setTimeout(()=>{
        throw Error("122") //该错误会被抛出，不会被catch到
        // reject("00")// 替换上一行，则也catch不到该错误，也不会抛出，因为promise已经resolve
    },100);

    resolve("1");
}).then(res=>{
    console.log("正常执行")
    console.log(res);
}).catch(e=>{
    console.log("出错了") 
    console.log(e);
})
```

注：unhandled promiserejection 是不会被window.onerror捕捉到的，可以使用settimeout异步抛出。

```javascript
内层失败的信息，被外层捕获。过两秒，打印出 '2' 2000 。

new Promise((resolve, reject) => {
  resolve(createPromise());
}).then(res => {
  console.log('1', res);
}, err => {
  console.log('2', err);
});

function createPromise() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(2000);
    }, 2000);
  });
}
```

---

### 2.promise.all

> `**Promise.all(iterable)**` 方法返回一个 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 实例，此实例在 `iterable` 参数内所有的 `promise` 都“完成（resolved）”或参数中不包含 `promise` 时回调完成（resolve）；如果参数中  `promise` 有一个失败（rejected），此实例回调失败（reject），失败原因的是第一个失败 `promise` 的结果。

即哪一个先失败则返回哪一个promise的错误结果。且只返回一个。

如果我们需要多个函数并行执行，且不因为其中一个失败而中断，最好promise.all中的promise都能确保完成，然后将其错误信息保存起来。

```javascript
const promise1 = function() {
    return Promise.resolve("123");
};
const promise2 = function() {
    return Promise.reject("00");
}
const promise3 = function() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            reject("000000")
        }, 2000)
    })
}

function resolveAll(cbs, errors) {
    return cbs.map((f, index) => {
        return f().then(res => {
            return res;
        }).catch(e => {
            errors[index] = e;
        })
    })
}
const errors = [];
Promise.all(resolveAll([promise1, promise2, promise3], errors)).then(res => {
    console.log(res);
    console.log(errors)
})
```

---

### 3.promise.finally

和try catch finally一样，不论是onfulilled或onRejected都会执行，当然不过不catch住reject的错误，finally依旧执行，而错误依旧抛出，且不被onerror捕获。

### 4.trycatch

与promise一样，trycatch只能捕捉到第一层的promise的错误。

如

```javascript
async function main() {
			try {
                await pro1();
				await pro2()
			} catch (e) {
				setTimeout(()=>{
                    throw e;
                })
			}
		}
```

当pro1抛出错误，pro2也不会执行。

```js
async function main() {
			try {
                await pro1().then(()=>{
                    pro2();
                })
			} catch (e) {
				setTimeout(()=>{
                    throw e;
                })
			}
		}
```

运行到pro2报错时，不会被catch到，因为pro2在pro1的then中独立运行了，如果换成以下做法则可以获取到错误

```js
async function main() {
			try {
                await pro1().then(()=>{
                    return pro2();
                })
			} catch (e) {
				setTimeout(()=>{
                    throw e;
                })
			}
		}
```

因为pro2返回的promise又由外层的await处理了。

*与promise一样，await中发生的错误时uncatched promiseerror，不被window.onerror得到，还是需要在catch中settimeout抛出错误*

