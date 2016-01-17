title: '[RN]1.环境搭建--React-Native learning notes'
date: 2015-11-21 20:57:35
tags:
- React-Native
- learning notes
- FE
categories: [React-Native]
---
# tip:
这是一篇学习 React-Native for Android 的学习笔记，主要上其实是看官方的文档来自己学习 RN，目前的主要是学习android的开发。

# React-Native for Android 的环境搭建
### 要求：
1. 前提 OS X (假设大家有用Mac了😏ß);
2. [Homebrew](http://brew.sh/) (包管理器)(用来安装Watchman 和 Flow);
3. 安装[node.js](https://nodejs.org/) 4.0 或者以上;
4. 安装 watchman(文件监控) ,在命令行: `brew install watchman`;
5. 安装 flow ,在命令行: `brew install flow`;
6. 安装最新的 [JDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html);
7. 安装 SDK ,在命令行: `brew install android-idk`;
8. 配置 android 环境:
在 ~/.bashrc, ~/.bash_profile写入
     `export ANDROID_HOME=/usr/local/opt/android-sdk`;
(在这一步可以先到~目录下，然后如果没有.bashrc，就创建一个，然后vim .bashrc，然后写入上面代码，按esc键，输入`:wq`，然后自动保存退出，.bash_profile同理);
9. 配置 SDK:
新开一个shell窗口：运行 `android`,
会弹出 Android SDK Manager窗口，
安装下列选项:
![](/img/RN_img_1.png)
![](/img/RN_img_2.png)
(tip: 这一步可能需要翻墙);
10. 安装 [Genymotion](https://www.genymotion.com/#!/download/freemium/mac/classical) (android 的模拟器,推荐使用这个吧);
11. 安装react native 的命令行工具包,在命令行: `npm install -g react-native-cli`;
12. 命令行进入需要创建的文件夹目录下: `react-native init AwesomeProject`，会自动获取下载react native的包（建议翻墙）;
13. 项目的目录结构如下：
```
.
├── android           #android需要的文件夹
├── ios               #ios需要的文件夹
├── node_modules      #npm包文件夹
├── index.android.js  #android应用的入口js文件
├── index.ios.js      #ios应用的入口js文件
└── package.json
```

14. 打开Genymotion,下载安装最新的android模拟器，打开;
15. 在文件夹下运行命令行: `react-native run-android`,等待安装，
然后android的模拟器就会出现：
![](/img/RN_img_3.png)
ß
## 完成！
