Promise手动终止封装

# 方案一

```js
function wrap(promise) {//真正的Promise
    let abort = null;
    let res = null;
    let _p = new Promise((resolve, reject) => { //壳子Promise
      res = resolve;//调用这个函数是的 _p的状态变为成功
      abort = reject;//调用这个函数是的 _p的状态变为失败
    });
    _p.abort = abort;
    
    promise.then(res, abort);
    //真正的Promise状态变为成功，就会触发壳子Promise成功
    //真正的Promise状态变为失败，就会触发壳子Promise失败
    
    return _p;//壳子Promise
  }
let p = wrap(new Promise((resolve,reject)=>{
setTimeout(() => {
    resolve(1);  
}, 3000);
}))
p.then((data)=>{console.log(data)},()=>{console.log('失败')})
p.abort()；//手动指定状态为失败，其实是主动调用了 reject 方法

```

当手动调用 `abort` 时，壳子 `Promise`状态变为失败，这个时候真正的 `Promise`的任务仍然在继续，当真正的 `Promise`的任务成功时，也会**尝试**使得壳子 `Promise`状态变为成功，但是一个 `Promise`的状态一旦确定了就不会发生改变。

> - 这个方法本质是将 `promise`进行包装，在内部再建一个 `promise`,把 `resolve|reject`方法单独提取出来，传给真实的 `promise的 `then方法上.
> - 这个 `p.then`调用的是内部 `Promise`,也就是通过 `abort`来直接执行 `reject`，真正的 `promise`因为执行了 ·`reject`,导致状态已变更。
> - 但是这个有个缺点，真正的 `Promise`虽然状态被改掉了，但是实际还是得等异步主体函数运行等待完才完成。

# 方案二

```js
function wrap(promise) {
            let res, abort;
            let _p = new Promise((resolve, reject) => {
                res = resolve;
                abort = reject;
            })
            _p.abort = abort;
            promise.then(res, abort)
            Promise.all([promise, _p]).then(res, abort);
            return _p;
        }
        let p = wrap(new Promise((resolve, reject) => {
            setTimeout(() => {
                resolve(1);
            }, 3000);
        }))
        p.then((data) => {
            console.log(data);
            console.log('外部成功')
        }, (data) => {
            console.log(data);
            console.log('外部失败')
        })
        p.abort(2)
```

