# ReactNative-notes
RN项目笔记


###duplicate/undefined symbols for architecture x86_64

1. xcode出现undefined symbols for architecture x86_64。尝试删除项目的Derived Data并重新编译。Window / Projects / Derived Data -> Delete。[问题链接](http://stackoverflow.com/questions/18408531/xcode-build-failure-undefined-symbols-for-architecture-x86-64)。

2. xcode出现undefined symbols for architecture x86_64。可能是重复引入了类库，检查xcode项目文件结构根目录和libraries目录以及其他目录下是否引入了相同的库文件。

###ScrollView组件的scrollResponderZoomTo方法
ScrollView组件有许多内置方法，查看其实例可看到。其中`scrollResponderZoomTo`方法可以用于放大缩小ScrollView视图，做图片放大浏览插件的时候用得到（iOS支持）。`scrollResponderZoomTo(x:number,y:number,width:number,height:number,animated:boolean)`方法定义在ScrollResponder.js文件中。参数说明：width、height表示把ScrollView聚焦到宽高为width、height的小矩形里，和放大镜类似；x、y是小矩形的中心点。

###图片消失问题
页面中存在引用本地图片的Image组件时，如果此时调用native modules，比如native的相册和照相机，图片会消失再出现。出现这种问题的条件是Image的source使用了本地图片。根本原因还不知道。暂时的解决办法是引用本地图片的Image组件必须设置defaultSource属性（defaultSource属性只支持local地址），source可选设置（android上必选，否则图片不显示）。

###升级RN
一些问题可以参考[这里](http://blog.csdn.net/xiaominghimi/article/details/51565423)。[中文文档](http://reactnative.cn/docs/0.26/upgrading.html#content)里的步骤是修改package.json里的react-native版本号然后install就行，这样可能导致react版本不对应的提示。
可以删除package.json里的react-native和react记录，再npm install react-native@latest -save，完成后，如果react版本还不对，可以手动npm install react@xxx -save 安装对应的版本。注意，package.json里的react版本记录可能是类似’^15.0.2’，而react-native可能需要的是~15.0.2，可直接把^ 改成~。版本install没问题后，xcode里：window里的projects下，delete对应项目的derived data后，重新编译。安装的第三方模块很可能过时，注意更新。[新的文档](http://facebook.github.io/react-native/docs/upgrading.html)已经改变了升级的步骤。

###键盘事件
iOS的键盘事件用keyboardWillShow和keyboardWillHide;

android的键盘事件用keyboardDidShow和keyboardDidHide。



###动画失帧问题
js线程繁忙时会导致动画失帧，比如执行http请求。最好是等动画结束后才调用http请求。解决的方法有两种：


1、在动画结束回调中emit消息。比如在Navigator的场景切换完成后emit一个完成消息，当前页面接收到消息后开始处理繁杂逻辑。
 
 ```
 <Navigator onDidFocus={this.sceneTransitionDone.bind(self)} />

 //sceneTransitionDone方法
 sceneTransitionDone(route) {
     DeviceEventEmitter.emit('sceneTransitionDone', route);
 }
```
然后，需要执行复杂js的页面处理接收消息：

```
DeviceEventEmitter.once('sceneTransitionDone', function () {
//do some heavy task
});
```

2、使用内置的InteractionManager。在需要执行复杂js的页面：

```
componentDidMount(){
	InteractionManager.runAfterInteractions(()=>{
		//do some heavy task
	})
}
```

方法一使用场景比较有限，如果要监控多个不相关的动画，就要分别添加动画完成的回调。不过也正好能够实现对不同动画结束后的独立回调需求，还可以自定义携带的回调参数；

方法二使用RN提供的延迟计划函数，能够更加精确和有效的执行延时操作任务。不过相比于方法一，方法二在某些场景下不能满足需求。比如返回到前一个页面的动画结束后需要触发刷新的需求，方法二并不能触发（因为页面在首次进入时已经触发方法二设置的延时函数了，所以不再触发？）。

###组件细化
需要进行频繁交互的组件尽量细分化，避免不必要的交互的部分也被频繁更新。比如上传组件里的进度条可以单独做成一个组件。


