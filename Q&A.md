* Q : 异步转同步是如何实现的？通过下面的方法为啥就能立刻得到数据？  
![](./images/1.png)  
A : 这是 Unity 的 API 的特性，异步加载没有完成的时候，如果访问了这个属性就会转为同步。
会让原有的异步加载立即完成，这是异步转同步的奥秘，这样异步可以当做同步来使用了。  
![](./images/2.png)  

* Q : 打包scene的时候会自动把scene里引用的资源打进去对吧，如果scene上挂了脚本，这个脚本会打进去吗，有些脚本没有挂在scene上会被打进去吗？  
A : bundle 里面不能打包代码，代码保存的是一些序列化信息。gameObject 的脚本是反序列化出来的。
![](./images/3.png)  

* Q : xasset是如何解决重复加载和循环依赖的问题的？  
A : 循环依赖是代码分层问题,7.0 的依赖通过 Dependencies 类管理的。资源的依赖归属于 Asset，不归属于 Bundle。结构类似于下面：  
```
Asset
 - Dependencies
```
不是下面的这种结构：
```
Bundle
 - Dependencies
```
加载 Dependencies 的时候没有递归,不会死循环。是通过分层处理的，依赖关系放到 Asset 层，不是放到 Bundle 层。xasset的设计是 Bundle 自身没有依赖，Asset 有依赖，用 Asset 管理依赖，而不是用 Bundle 管理依赖，你看看 Bundle 类，有持有 Dependencies 么？分层就是不用 Bundle 管理依赖，因为依赖在 Bundle 层，天生有循序依赖，而依赖在 Asset 层，天生没有循环依赖。  

Q : 请问怎么批量加载？  
A : 你对批量加载如何定义呢?onebyone 还是 并行？这都是在业务层控制的。  
```C#
foreach(var asset : assets) {
    var asset = Asset.Load(asset, assetType);
    yield return asset; 
}
```
上面这种是这是 onebyone 加载。  
```C#
var assets = new List<Asset>();
foreach(var path in assetPaths) {
    assets.Add(Asset.LoadAsync(path, type));
}
yield return new WaitForComplete(assets.TrueForAll(a => a.isDone));
```
这种写法是异步并行加载。  
![](./images/4.png)  