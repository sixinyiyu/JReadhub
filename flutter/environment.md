
### 下载、配置```Flutter```

先下载```flutter SDK```[官网地址](https://flutter.io/docs/development/tools/sdk/archive#windows)

我这里下载的是1.0稳定版

因为国内的情况在没梯子的情况下，我们需要配置```Flutter```镜像地址

配置镜像地址

```shell
FLUTTER_STORAGE_BASE_URL   https://storage.flutter-io.cn 

PUB_HOSTED_URL 	https://pub.flutter-io.cn
```

配置```Flutter```环境变量
解压后将flutter/bin目录添加到path中去


### 下载安装并配置```Android Studio```

![下载Studio](http://wx1.sinaimg.cn/large/9c349f47gy1fxxe6uv8mmj210y04at8m.jpg)
速度太慢了我就用我国外的服务器下载的在传到本机，下载完后就安装罗就不说了


安装SDK，这里下载也非常缓慢可以慢慢的刷下B站
![下载SDK](http://wx2.sinaimg.cn/large/9c349f47gy1fxxe5ykxmqj20fx0a73yo.jpg)

终于安装完成了，重启并配置```Android HOME```环境变量
![下载SDK完毕](http://wx1.sinaimg.cn/large/9c349f47gy1fxxe62bn3vj20jw0bnjrz.jpg)

安装```Flutter```插件
![开始安装插件](http://wx3.sinaimg.cn/large/9c349f47gy1fxxe5degrhj20ie0f3aaj.jpg)

![Flutter插件](http://wx2.sinaimg.cn/large/9c349f47gy1fxxe5gxa1oj20l50fqmy2.jpg)

安装模拟器

在```Studio```中首先新建一个模拟器

![新建模拟器](http://wx4.sinaimg.cn/large/9c349f47gy1fxyhgd5fyxj20rj0hmgnk.jpg)

```Studio``` 实在是太卡了，我还是用VS或者Intej开发算了，暂时只需要一个模拟器就可以了，把启动模拟器的命令做成一个脚本

```shell
D:\code-mess\Android\Sdk\emulator\emulator.exe -netdelay none -netspeed full -avd Pixel_2_API_28

Pixel_2_API_28 为模拟器名(新建模拟器的时候取得)
```

启动模拟器的时候报错
```
emulator: ERROR: x86 emulation currently requires hardware acceleration! Please ensure Intel HAXM is
```
![说是需要硬件加速InteHAXM](http://wx4.sinaimg.cn/large/9c349f47gy1fxyi2weqt9j20z20czmy4.jpg)

此外还需要去bios中开启```Virtualization Technology```,将此菜单下的选项全部```ENABLE```

![Virtualization Technology](http://wx1.sinaimg.cn/large/9c349f47gy1fxyi668e28j20l306p10o.jpg)


### 配置```VS Code```

直接搜索```Flutter```然后安装后重启就可以了

### 验证

```shell
flutter doctor

```

![未安装licenses](http://wx1.sinaimg.cn/large/9c349f47gy1fxxe51q3snj20hd05va9w.jpg)

提示有2个感叹号(警告),一个是SDK证书问题，一个是未找到设备(模拟器)

![验证成功](http://wx2.sinaimg.cn/large/9c349f47gy1fxxe59byflj20l009874f.jpg)

安装完模拟器后再次验证

![启动模拟器后验证](http://wx4.sinaimg.cn/large/9c349f47gy1fxyhgj0uuxj20he05imx3.jpg)


### 启动项目


执行```flutter run```

:broken_heart:

![镜像](http://wx1.sinaimg.cn/large/9c349f47gy1fxyhgmsbb6j20po0bsgma.jpg)

修改android目录下```build.gradle```仓库地址后再次运行


```
    repositories {
        // google()
        // jcenter()
        maven{ url 'https://maven.aliyun.com/repository/google' }
        maven{ url 'https://maven.aliyun.com/repository/jcenter' }
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public'}
    }
```

![镜像报错](http://wx2.sinaimg.cn/large/9c349f47gy1fxyhgrwg3vj20qy0dkq3n.jpg)

发现还想仓库地址的问题，再修改```flutter```全局地址

![全局镜像](http://wx1.sinaimg.cn/large/9c349f47gy1fxyhul96z7j209l0833yd.jpg)

![全局仓库](http://wx2.sinaimg.cn/large/9c349f47gy1fxyhgxfbwjj210h0g90vf.jpg)

:clap: 恭喜!!!!项目终于跑起来了