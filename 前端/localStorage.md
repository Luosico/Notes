# localStorage

​	在HTML5中，加入了一个localStorage特性，这个特性主要是用来作为本地存储来使用的，**解决了cookie存储空间不足的问题**(cookie中每条cookie的存储空间为4k)，localStorage中一般浏览器支持的是5M大小，这个在不同的浏览器中localStorage会有所不同。它只能存储字符串格式的数据，所以最好在每次存储时把数据转换成json格式，取出的时候再转换回来。

​	除非清除浏览器缓存，或者主动清除，否则会一直存在



## 用法

```javascript
var name="hello";

//设置值
localStorage.setItem("name",name);
//覆盖之间的值
localStorage.setItem("name","world");

//获取值，若不存在返回 null
var name1 = localStorage.getItem("name");

//删除值
localStorage.removeItem("name");

//清除localStorage中所有信息
localStorage.clear();

//获取键
for(var i=0;i<storage.length;i++){
	var key=storage.key(i);
    console.log(key);
}
```

