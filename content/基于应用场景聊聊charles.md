> 在我从开发web变成开发wap之后，wap上的调试没有以前那么方便了，然而有个工具对我在调试的帮助非常大，那就是charles(或者finder)。


除了开发之外，测试大佬在测试wap的时候用charles也是很好用的，在很多时候，测试反馈了一些问题之后，开发要做的很多时候就是手机连上代理，然后按照测试大佬说的操作场景去复现，当然也会关注charles里面的信息，例如：

1. 静态资源是否有缓存
2. 是否因为接口数据返回的问题，这个时候就能确定是自己的问题还是后端接口的问题

### 手机连接代理
1. 代理手机看请求
查看ip（直接用终端查也是可以了，ipconfig[win系统] 或者 ifconfig[mac可以用这个]）,
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/1.png)

默认端口是 8888

如果 8888 端口被占用的话，那么可以proxy settings 面板调整port
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/2.png)
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/3.png)
在设置好代理之后，就可以用手机来连接代理了，在手机连接代理后会弹窗提示要不要允许连代理，点击allow按钮，同意连接。
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/4.png)
2. 在代理手机之后，在抓请求的时候，会遇到https请求的话抓取不了的问题，是提示“unknow”,具体表现如下：

那么这时候要做的有两步，
a. 给手机安装证书；
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/5.png)
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/6.png)


直接在手机自带浏览器上输入 chls.pro/ssl 就可以了，根据不同的系统，安装证书的步骤也有点区别
android:（小米安装不了，这个后期会整理一下看有没有其他办法，而且android在系统升级了之后，也有可能安装证书的方式变了，例如我自个的荣耀机子）

->打开自带浏览器，输入chls.pro/ssl,进行证书下载
->根据提示，安装证书，应该需要输入锁屏密码，如果没有密码的话，需要设置之后才能安装
ios：
->打开Safari，输入chls.pro/ssl,进行证书下载
->提示安装
->安装成功后，
进入 设置
->关于手机
->证书权限，打开对应的证书信任开关

b. 设置SSL Proxying
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/7.png)
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/8.png)



在 SSL Proxying 面板添加对应的设置。这里提一下，如果是写指定的域名的话，那么可能每有一个https的域名，你都得添加一次，有一种可以偷懒的方法，那就是直接添加一条
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/9.png)

做完上面这两步之后，你就可以愉快的通过手机代理进行抓请求了。（这个时候其实也有可能会出现请求是 unknown的一个情况，例如域名本身的证书过期，或者像andoid在打包的时候选择的sdk是android 6.x 的版本，会发现抓不了https，需要客户端的同事在debug包做些处理。）
现在已经可以抓请求调试了，但是光是这样还是满足不了日常的一些调试场景，例如有时候在修复bug的时候每次都得打包好静态资源，然后上传到对应的环境，然后再进行验证，如果验证有问题的话，还得重复这样去操作，这样是很费时间的。（比较有些项目只能是在特定的环境调试，没办法在本地调试）

### 静态资源代理
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/11.png)
这里的方式有两种
1. Map Remote 指定对应链接目录，例如
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/12.png)
把所有https://xxxx.com/static/下面的资源，直接访问到 http://192.168.10.17:8080/dist/下面对应的资源，可以通过这种方式在本地去调试。

2. Map Local	指定对应本地目录，例如
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/13.png)

把所有https://xxxx.com/static/下面的资源，直接访问到本地目录下面对应的资源。

#### 应用场景
1. 代理到本地资源，方便调试bug
2. 在测试环境被占用，且只是前端的测试环境被占用，那么可以通过以上方式来达到测试的效果，当然到最后，肯定也是要在正式环境里面过一遍的。

3. 接口数据更改
有时候在测试的时候，需要测一下接口返回特殊数据的情况，而且是比较难以达成的条件，例如，等级要到达9999级的时候才触发，这时候有个方法就是直接修改接口返回对应特殊的数据来测试。

在左边请求显示栏目，右键，然后点击 breakpoints ，断点这个请求


当第二次请求的时候，就会来到这个界面，编辑request信息，点击 execute，进入下一步
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/14.png)
点击编辑 editResponse
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/15.png)
这个时候直接修改json，再次点击 execute 就完成了请求。返回结果
![image](https://github.com/hanyitim/Blog/blob/master/static/images/charles/16.png)


后面再继续补充。有错误的地方，欢迎指点。
