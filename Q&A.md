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

* Q : 请问怎么批量加载？  
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

* Q : 请问怎么打android包，怎么把shader打进去？  
A : 1.切换平台到 Android  
2.Assets/Versions/Build/Player 或者 File/Build XXX 都能打包 Android 包  
3.shader 分为两种，Unity 内部的配置到 allways include，其它的设置下 AssetBundle 名字  

* Q : 请问打包到安卓手机上，3d物体是紫色的什么情况?  
  A : shader 没有打包，或者变体采集，还有兼容性问题。shader 的问题，用 FrameDebuger 很好定位  

* Q:![](./images/5.jpg)

  对于Build的Group拆分有什么建议吗？比如
  1.打包后，对大小超过多少的AB进行拆分成按文件打包？
  2.对于不同父节点的同名的文件夹如何处理？非PackTogether的情况下，Group.name是否可以重复？
  3.哪类文件适合单独一个ab，例如scene、所有的Shader
  4.在不修改当前Group的情况下，如何把Shader抽出来放到一个ab里(ustomPacker += CustomPacker实现)？

  A:

  1.prefab、场景 之类可以 按文件打包。贴图之类 可以按目录打包、一个bundle 建议控制在 5MB 以内

  2.不同节点的同名的文件夹 不是 PackTogether 的 BundleMode 没影响。PackTogether 的建议不要重名，PackTogether 是根据 group.name 为采集的资源分配 ab。

  3.prefab、scene 之类比较适合 一个文件一个 ab

  4.不修改 Group 除了 CustomPacker 外还可以 修改资源目录实现 把shader 抽出来放到 一个 ab,shader 的最优打策略不是全部放到一个 ab,具体看项目体量了,所有资源都可以遵循 同时使用的 放到一起打包