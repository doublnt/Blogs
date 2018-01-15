##Ecmascript6 入门---Promise

容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。

* 对象的状态不受外界影响。Promise 对象代表一个异步操作，有三种状态：pending 、fullfilled、rejected。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。
* 一旦状态改变，就不会再变，任何时候都可以得到这个结果。Promise对象的状态改变，只有两种可能： pending -> fullfiled，pending -> rejected。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，此时称为  resolved（已定型）

Promise 的缺点，首先，无法取消 Promise，一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，Promise 内部抛出的错误，不会反应到外部。第三，当处于pending 状态时，无法得知目前进展到哪一个阶段。

```javascript
//Promise 的一个简单例子
function timeout(ms){
 return new Promise((resolve,reject)=>{
   setTimeout(resolve,ms,"停止");
 });
}
timeout(1000).then((value)=>{
  console.log(value);//过一秒后才执行 timeout 函数
});


```

```javascript
const getJSON = function(url){
  const promise = new Promise(function(resolve,reject){
    const handler = function(){
      if(this.readyState !== 4){
        return;
      }
      if(this.status === 200){
        resolve(this.response);
      }else{
        reject(new Error(this.statusText));
      }
    };
    const client = new XMLHttpRequest();
    client.open("GET",url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept","application/json");
    client.send();
  });
  return promise;
};
getJSON("/posts.json").then(function(json){
  console.log("Contents:"+json);
},function(error){
  console.error('Error',error);
});
```

