---
layout: post
author: syea
title: JavaScript自动获取Reveal体验码
# subtitle:
# header-img: 
# header-mask:  
# catalog: 
# multilingual: 
tags:
    - JavaScript
---

# JavaScript自动获取Reveal体验码

#### 起因

在网上搜了一圈没有找到 Reveal 的最新破解版(难为情..下意识的就想找破解版)，在 Reveal 官网[https://revealapp.com/download/](https://revealapp.com/download/) 可以通过输入邮箱、first name、last name 来获取 Reveal 的 `Activation Number`，通过这个`Activation Number` 可以进行为期14天的试用。由此想到，只要有无限的邮箱就能无限地试用。

#### 分析

1. 个人的邮箱数量有限，如何获取无限的邮箱？
在搜索栏输入临时邮箱，结果还真的有好多。经过筛选（免费、方便...）此处我选择了[https://www.guerrillamail.com/](https://www.guerrillamail.com/)。

2. 如何向目标网页注入 js ？
利用 Chrome！教程网上有很多。

接下来确定业务流程:<br>
**前往临时邮箱网站**->**将生成的邮箱填入reveal申请界面**->**返回临时邮箱网站获取`Activation Number`**

ok, 准备实践。

#### JavaScript实践

1. 编写Chrome插件配置文件
```
{  
  "name": "Chorme自定义脚本插件",  
  "manifest_version": 2,  
  "version": "1.0",  
  "description": "测试 js 脚本",  
  "browser_action": {  
    "default_icon": "icon.png"  
  } , 
  "content_scripts": [  
    {  
      "matches": ["https://revealapp.com/download/*","https://www.guerrillamail.com/*"],  
      "js": ["getRevealCode.js","getRandomTempEmail.js"]  
    }  
  ]  
} 
```

表示Chrome在跳转至`https://revealapp.com/download/*`以及`https://www.guerrillamail.com/*`时会分别注入`getRevealCode.js`和`getRandomTempEmail.js`。

2. 编写JavaScript

按照流程，先前往临时邮箱网站获取临时邮箱
```
var tempEmail = document.getElementById("email-widget").innerText;
```

然后前往 reveal 官网,并将生产的临时邮箱做为参数传过去，传参我查了很多种方法，有的在 url 后加 ？=xxx 进行传参，有的用 post 进行传参，我试了几次都没有成功，可能是我道行还不够。后来自己想了一下加了 # 进行传参，因为加 # 对 url 的跳转没有任何影响。
```
window.location.href = "https://revealapp.com/download/#email=" + tempEmail;
```

`getRandomTempEmail.js` over。

---

然后到 reveal 官网获取数据并填写，然后发送申请。
```
var url = window.location.href;
var tempEmail = url.substring(url.indexOf("#") + 7,url.length);

function setParam (email) {
    document.getElementById("email").value = email;
    document.getElementById("first_name").value = email.substring(0,2);
    document.getElementById("last_name").value = email.substring(2,4);
}

function getCode (){
    document.getElementById("trial-license-request").submit();
}

setParam(tempEmail);
getCode();
```

此时应该跳回邮箱界面，无奈一跳回邮箱界面就会执行刚才的 js 形成一个死循环。于是我跳转了[https://www.sharklasers.com/](https://www.sharklasers.com/)。这两个网站好像是一家的 = =！，只是域名不同，刚好解决了死循环的问题。
```
window.open("https://www.sharklasers.com/" + "#" + tempEmail);
```

`getRevealCode.js` over。

#### 总结

看起来的写的很简单，实际上做的时候碰了很多坑，各种调试。最后也不是很完美，返回邮箱去获取`Activation Number`的时候还得输入一开始获取的tempEmail，并等待邮件的到来，再阅读邮件获取`Activation Number`。

总的来说还是一次不错的尝试.