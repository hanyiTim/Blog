> 在实践过bodymovin之后，发现如果需要在动画里面加逻辑比较麻烦，效果也不好。因此后面找到了另外一个开源的工具，就是YY开源出来的[svga](https://github.com/yyued/SVGAPlayer-Web),有android，ios，web版本，还有对应的AE插件，用法请看官方文档

之所以选择svga：
1. 比较强大的api；
2. 据说性能比bodymovin好（不敢下定论，因为按照目前接触过bodymovin跟svga的情况来看，感觉如果只是纯播放，bodymovin的性能会流畅一点，当然，这个跟我的应用场景有关系，一个比较蛋疼的原因，就是需要在本地播放。意味着动画资源不能通过ajax获取）；

问题：因为需要本地播放，所以Parse.load传的是svga文件转成base64之后的字符串，load会多了一步readBlobAsArrayBuffer的过程，这个很耗浏览器性能。
##### 相关资源
* [SVGA-AEConverter](https://github.com/yyued/SVGA-AEConverter)(AE动画导出svga插件)
* [SVGAPlayer-Android](https://github.com/yyued/SVGAPlayer-Android)(android播放器)
* [SVGAPlayer-Web](https://github.com/yyued/SVGAPlayer-Web)(web播放器，本次要实践的库)
* [SVGAPlayer-iOS](https://github.com/yyued/SVGAPlayer-iOS)(ios播放器)

##### 常用api
* setImage(url,imagekey)
* setText(text || option,imagekey)
* startAnimationWithRangeList(自己添加的一个方法，指定区间帧动画，对区间动画进行拼接，全部执行完才调用 onfinish)

##### 简述用法
```
//安装svga
//script
<script src="https://cdn.jsdelivr.net/npm/svgaplayerweb@2.1.0/build/svga.min.js"></script>
//npm
npm install svgaplayerweb --save
添加 require('svgaplayerweb') 至 xxx.js

//使用
//自动加载(基于dom）
<div src="rose_2.0.0.svga" loops="0" clearsAfterStop="true" style="styles..."></div>
这里其实是跑一个autoplayer的一个方法，通过解析dom的一些信息，自动去做播放（基本没用过，因为，基本用到这个东西，都有一下逻辑在里面）

//手动加载（通过svgaPlayer的一些方法去播放）
1.添加容器
<div id="demoCanvas" style="styles..."></div>

2.加载动画
var player = new SVGA.Player('#demoCanvas');
var parser = new SVGA.Parser('#demoCanvas'); // 如果你需要支持 IE6+，那么必须把同样的选择器传给 Parser。
parser.load('rose_2.0.0.svga', function(videoItem) {
    player.setVideoItem(videoItem);
    player.startAnimation();
})

```
##### 需要注意的地方
1. android 4.x 需要导入blob，以及 jszip库。
2. 目前官方的库，提供的适配属性在iphone上有点问题。
##### svga动画信息
设计通过AE+svga插件，导出一个动画的svga格式的一个二进制文件，虽然看不到里面的内容，但是把他扔到[svga预览](http://svga.io/svga-preview.html)，是可以看到svga文件的一些信息的（转换之后）
```
svga基本信息
{
  "version": "2.0",
  "FPS": 30,
  "frames": 110,
  "videoSize": {
    "width": 750,
    "height": 750
  }
}


//素材信息
A --- {"width":600,"height":493}
Aleft --- {"width":82,"height":67}
Aright --- {"width":82,"height":67}
Avatar --- {"width":72,"height":72}
BG --- {"width":750,"height":750}
Bleft --- {"width":120,"height":97}
Bright --- {"width":120,"height":97}
Planet --- {"width":80,"height":60}
flower --- {"width":63,"height":26}
hole --- {"width":113,"height":109}
img_491 --- {"width":38,"height":38}
medal --- {"width":376,"height":415}
medal-bling-l --- {"width":66,"height":66}

```
当然，文件里面的信息肯定不只包含上面的信息，看svga-web的源码，里面的reader那一部分的逻辑，应该是有包含具体到哪一帧用canvas绘制上面样的内容的一些信息（这个好像是废话）。

##### 需求：宝箱1.0
设计大佬会给一个宝箱的svga，这边需要做的就是通过设计大佬给一个imagekey+svga的setImage去设置宝箱开处理的礼物图片。
##### 动画gif如下：
![image](https://github.com/hanyiTim/Blog/blob/master/static/images/animation-4.gif)
##### 部分代码如下：
```
var parser = new svgaplayerweb.Parser('#canvas')
var player = new svgaplayerweb.Player('#canvas')

//之所以用这个方法去做适配是因为官方的库设置fillMode属性有在ios7p/8p 上面有bug，那时候没去提issuse，不知道现在修复了没
function setFill(){
    var $_canvas = document.getElementById('canvas'),
        w = window.innerWidth,
        h = window.innerHeight,
        screen_proportion = h/w,
        svga_proportion = 16/9;
    if(screen_proportion > svga_proportion){//长屏幕
        $_canvas.style.width = h/svga_proportion+'px';
        $_canvas.style.left = (w-h/svga_proportion)/2+'px';
    }else{
        $_canvas.style.height = w*svga_proportion+'px';
        $_canvas.style.top = (h-w*svga_proportion)/2+'px';
    }
    
}

//初始化
function svgaInitial(giftSrc){
    // svga ready
    parser.load(svgabase64,function(videoItem){
        //设置宝箱开出的礼物图片
        player.setImage(giftSrc, 'gift')
        player.loops = 1;
        player.setVideoItem(videoItem);
        player.onFinished(function(){
            //播放结束后触发离开逻辑
            leave();
        })
        enter();
    },function(err){
        // alert(err.message);
        //报错直接触发离开逻辑
        leave();
    })
}
```

##### 需求：宝箱1.1
宝箱部分礼物需要开出图片，一部分需要开出特效，这个时候就比较为难，最后决定的解决方案是在宝箱里面放置了好几段特效，分别是：宝箱打开+普通礼物（1.0）+宝箱打开+大礼物特效*N（这样导致了一个特效包最大的差不多2M）
##### 动画gif如下：
![image](https://github.com/hanyiTim/Blog/blob/master/static/images/animation-4.gif)

![image](https://github.com/hanyiTim/Blog/blob/master/static/images/animation-5.gif)
##### 部分代码如下：
```
//相对于1.0的版本，1.1的版本多了好几个svga文件，多了以下判断逻辑，通过参数去觉得播放哪一个svga
if(giftId == 7){
    svgabase64 = box_JL;
}
else if(giftId == 8){
    svgabase64 = box_HD;
    
}
else if(giftId == 9){
    svgabase64 = box_CS;
}

问题：虽然只load一份base64，不过几个base64文件打包到一个js里面去，会导致js的体积很大
```

##### 需求：宝箱1.2
宝箱需要开出的礼物有不同的类别（最大五种）+不同的数量，如果这个效果按照1.1的形式去做的话，得需要有五段不一样的svga，那么整个特效包的大小肯定会超过2M。在去看文档的时候，发现了startAnimationWithRange这个方法，播放svga动画的某一个区间的动画，但还是满足不了，无奈之下fork一下仓库，自己实现一些功能
1. 多段拼接播放（startAnimationWithRangeDouble）
2. 特殊字体，字体颜色渐变，阴影的支持

##### 动画gif如下：

![image](https://github.com/hanyiTim/Blog/blob/master/static/images/animation-1.gif)

![image](https://github.com/hanyiTim/Blog/blob/master/static/images/animation-2.gif)

![image](https://github.com/hanyiTim/Blog/blob/master/static/images/animation-3.gif)
```
player.setText({
    text:form,
    size:form > 99 ? "160px" :"200px",
    color:"linear-gradient(to right, #e1d7b4, #bfae7d)",
    family:"Impact",
    offset:{
        x:0,
        y:-10
    },
    textShadow:{
        color:'rgba(0, 0, 0, 0.01)',
        offsetX:0,
        offsetY:3,
        blur:3
    }
},"numberone");
```

##### 动画gif如下：
##### 部分代码如下：
```
function svgaInitial(){
    var len = gift.length;
    var releaseGift = function(data,len){
        data.forEach((item,index) => {
            player.setImage(item.src,`gift-${len == 1 ? "" : len+"-"}${index+1}`);
            player.setText({
                text:`X${item.count}`,
                size:item.count>999 ? "40px":"54px",
                family:"BaiZhouRenZhe",
                color: "#FFF2C8",
                offset: {x: 0, y: 0}
            },`gift-count-${len == 1 ? "" : len+"-"}${index+1}`)
        })
    }
    // svga ready
    parser.load(svgabase64,function(videoItem){
        player.loops = 1;
        releaseGift(gift,giftLen);
        player.setVideoItem(videoItem);
        player.onFinished(function(){
            leave();
        })
        if(len > 0){
            enter();
        }
    },function(err){
        leave();
    })
}


function enter() {
    var releaseSvga = function(gift,len){
        switch(len){
            case 1:
                player.startAnimationWithRange({location:0,length:200});
            break;
            case 2:
                player.startAnimationWithRangeDouble([{location:0,length:150},{location:200,length:50}]);
            break;
            case 3:
                player.startAnimationWithRangeDouble([{location:0,length:150},{location:250,length:50}]);
            break;
            case 4:
                player.startAnimationWithRangeDouble([{location:0,length:150},{location:300,length:50}]);
            break;
            case 5:
                player.startAnimationWithRangeDouble([{location:0,length:150},{location:350,length:50}]);
            break;
        }
    }
    releaseSvga(gift,giftLen)
}

//相对于1.1的版本，这次1.2的版本减少了打包之后整体的体积，但是因为是全部动画都在一个svga文件，单个的svga的文件比较大，因此load起来的速度也会慢一些。
```
#### 总结
1. 关于bodymovin 跟 svga的性能，网上有分析，不过我还是觉得要自己分析一下才能下定论。
2. 由于需要本地播放，因此对我来说，svga2base64的体积比svga的体积会大25%左右，如果是bodymovin的话，一份json的体积其实还是挺小的，就是api支持不够
3. 对于android4.x的手机来说，不支持jszip，还有blob的，还得导入这两个库，这一部分的体积应该是在100kb左右，当然，这个是可以做成离线缓存的，这个又是另外一个话题了


bodymovin，svga帮助我们解脱了看着mov写特效，然后设计师坐在旁边花费很多时间去调试到满意的结果的日子。让前端可以更注重写逻辑这块的代码。






