## bodymovin实践
>需求：实现礼物特效，实现特效播放，连击逻辑，以及最终打包成本地文件，给到后台上传，用户会自动下载特效包到本地，在有特效的时候进行相关的特效播放


### 特效实现

 特效页面实现(部分代码)    [bodymovin的具体用法可以参考](https://github.com/bigxixi/bodymovin)
 ```
    import lottie from "lottie-web";
    var $_lottie = document.getElementById('vw_lottie'),
        isReady = false,
        animationData = require('./lottie.json');
    var frameCount = 125,    //一共多少帧，设计的动画一般1秒25帧
        maxFirstFrame = 25,       //连击快的时候，最大起始帧
        timeCount = 5*1000,    //动画时间，毫秒
        onFrameTime = timeCount/frameCount,     //一帧的时间
        lastTime = 0;
    var animation = lottie.loadAnimation({
        container:$_lottie,
        renderer:'canvas',
        loop:false,
        autoplay:false,
        animationData:animationData
    });
    export function lottieIsReady(callback){
        animation.addEventListener('DOMLoaded',function(){
            isReady = true;
            console.log('lottie is ready，准备好了');
            callback && callback.call && callback();
        })
    }
    export function lottieStart(firstFrame){
        console.log('lottieStart');
        if(isReady){
            firstFrame = firstFrame || 0
            lastTime = Date.now();
            animation.playSegments([firstFrame,frameCount],true);
        }
    }
    var timer = null
    export function lottieRepeatHit(){
        if(isReady){
            let now = Date.now(),
                timeDiff = now-lastTime,
                firstFrame = 0;
            firstFrame = maxFirstFrame - parseInt(timeDiff/onFrameTime);
            firstFrame = firstFrame >= 0 ? firstFrame : 0;
            lottieStart(firstFrame);
        }
    }
    
```
### 主要遇到的问题：
1. 连击的时候，特效是怎么展示
2. 特效的渲染方式，（bodymovin有三种渲染方式svg,html,canvas)，主要考虑的是在anroid的低端机子上的性能问题。
3. 因为是用户下载特效包到本地的，就是出现特效涉及的图片会在用户手机的相册里面出现，用户有可能误删的情况（直接影响特效展示），或者特效图片在相册这个也不是很美观
4. bodymovin是有load的，在加载动画json的时候，ajax load 文件只能是http or https协议头。

### 解决方案
1. 连击的处理；刚开始的处理是每次连击的时候，动画重新播放，那么点击得快的话，就会出现动画消失的情况（动画有入场），解决方案是通过计算连击的频率，频率越低，开始播放的地方就越趋向于0，频率越高，开始播放的地方就越趋向于动画完全呈现的位置。（入场 -> 完全显示 -> 离场）
2. 这里，目前的做法是用canvas感觉会好一点，有遇到问题就是用canvas渲染会模糊，改成svg渲染，结果在android的机子上很卡，最后改回来canvs，优化图片。
3. 动画特效相关图片会显示在相册的问题，用了webpack-replace-loader,webpack-file-loader,替换data.json里面的“png ”->"_ png _"，以及将图片的输出路径也改成一样的后缀
4. 在实例化Lottie的时候，不传path，用webpzck-json-Loader,把json转成对象传到animationData参数里面去。

### 小坑
* 在做其他特效的时候，遇到了一个需求是需要动态变化动画里面的一些元素，所以监听了complete 这个事件，这个事件有两个坑
    * 不是一帧才回调一次
    * 安卓或者性能差点的手机会出现掉帧的情况，也就是说如果针对某一帧去做一些操作的话，是有可能会执行不到的。