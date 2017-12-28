---
layout: post
author: syea
title: 尝试 ReactNative
# subtitle:
# header-img: 
# header-mask:  
# catalog: 
# multilingual: 
tags:
    - ReactNative
---
# ReactNative

## 1.环境搭建

ReactNative 中文: [http://reactnative.cn](http://reactnative.cn)

ReactNative 英文官网: [http://facebook.github.io/react-native/](http://facebook.github.io/react-native/)

官网都有详细的教程,有从零开始新建一个 reactnative 项目,也有讲如何接入原生项目。

## 2.实践

官网的教程中还是会有点小坑,Demo 跑不起来的情况下,建议 google 一波,或者 StackOverflow。

#### iOS 项目中需要的操作
1. 项目中的 podfile 添加
```
pod 'React', :path => './ReactNative/node_modules/react-native', :subspecs => [
'Core',
'BatchedBridge', #RN版本高于0.45之后必须导入
'DevSupport',
'RCTText',
'RCTNetwork',
'RCTWebSocket',  # 这个模块是用于调试功能的
]
pod 'yoga', :path => './ReactNative/node_modules/react-native/ReactCommon/yoga/yoga.podspec'
```

2. 创建要使用 reactnative 的模块
导入头文件

 `#import <React/RCTRootView.h>`
 
创建视图

```
    RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:[NSURL URLWithString:@"你的url"]
                                                        moduleName:@"你的模块名"
                                                 initialProperties:@{}
                                                     launchOptions: nil];
    [self.view addSubview:rootView];
    [rootView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.equalTo(self.view);
    }];
```

iOS端 Over

#### 后端需要的操作
1. 打开终端 cd 到 reactnative 路径下
2. tip:在 package.json 文件中可以改服务启动的端口号
```
	"scripts": {
		"start": "node node_modules/react-native/local-cli/cli.js start --port 3000",// 修改端口号
		"test": "jest"
	},
```
3. npm start

后端 Over

# 3.开始学着去控制 `ReactNative`

1. 首先，通过改写 `index.ios.js` 可以改变iOS端展示的界面
2. 其次，去学习如何写 react 吧 = =！  

加油！

