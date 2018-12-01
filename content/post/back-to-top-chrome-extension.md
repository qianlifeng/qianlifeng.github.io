---
title: 在chrome中利用“油猴”为每个网页增加“返回顶部”功能
date: 2012-03-08 13:46:00
tags: [javascript]
---

今天在网上看到了“油猴”这么个东西的介绍，感觉蛮有趣，跟大家分享一下。

这个东西本来是火狐的插件，后来一查原来 chrome 从 4.0 以后也支持用户脚本了。所谓的“油猴”实际上是一段以\*.user.js 结尾的 js 脚本，通过油猴你可以在每个网页上执行你的这段脚本。
利用这个 js 脚本你可以对网站页面做一些个性化的修改，比如像我们今天提到在每个页面上增加“返回顶部”的功能，其效果就是在每个页面自动添加返回顶部的功能：
<img src="/Images/back-to-top-chrome-extension/1.png"/>

# 创建“油猴”脚本

下面介绍一下如何在 chrome 中编写自己的用户脚本（以增加返回顶部功能为例）

1. 去谷歌扩展中心下载 Tampermonkey ,这个东西可以用来管理各种用户脚本。
2. 安装后进入 Tampermonkey 的设置界面，然后创建一个新的脚本：
   <img src="/Images/back-to-top-chrome-extension/2.png"/>  
   在编辑脚本页面输入下面的一段 js 脚本（用于实现“返回顶部”功能）

   ```
   // ==UserScript==
   // @name GotoTop
   // @version 0.1
   // @description 给每个网页增加返回顶部按钮
   // @match http://*
   // @match https://*
   // @copyright scott qian
   // @require http://ajax.googleapis.com/ajax/libs/jquery/1.6/jquery.min.js
   // ==/UserScript==

   ImportCss();
   ScriptWithJquery();
   BindHotKey();

   function ImportCss() {
       var jqueryScriptBlock = document.createElement('style');
       jqueryScriptBlock.type = 'text/css';
       jqueryScriptBlock.innerHTML = "#gototop{position:fixed;bottom:20%;right:1px;border:1px solid gray;padding:3px;width:12px;font-size:12px;cursor:pointer;border-radius: 3px;text-shadow: 1px 1px 3px #676767;}";
       document.getElementsByTagName('head')[0].appendChild(jqueryScriptBlock);
   }
   ```


    function ScriptWithJquery() {
         $(document.body).append("<div id='gototop' title='快捷键： alt + up alt+鼠标滚轮向上'> 返 回 顶 部 </div>");
         $('#gototop').click(function () { $('html,body').animate({ scrollTop: '0px' }, 800); return false; });
    }

    function BindHotKey(){
        document.onkeydown = function(){
            var a = window.event.keyCode;
            if((a == 38)&&(event.altKey))
            {
                //alt + up
                $('html,body').animate({ scrollTop: '0px' }, 800);
            }
        };

        //绑定alt+鼠标向上滚轮事件
        window.addEventListener('mousewheel', function(event){
            if(event.wheelDelta > 0 && event.altKey)
            {
                $('html,body').animate({ scrollTop: '0px' }, 800);
                //防止滚动条向上滚动，导致多重效果
                window.event.preventDefault();
            }
        }, false);
    }
    ```
    <img src="/Images/back-to-top-chrome-extension/3.png"/>

    这里有一个地方需要说明一下。为了方便js的开发，我这里使用了jquery，所以需要在头部声明中声明以下一段话。这样我们就可以在其中使用jquery了。
    ```
    // @require http://ajax.googleapis.com/ajax/libs/jquery/1.6/jquery.min.js
    ```

3. 保存该用户脚本然后随便打开一个网页，效果如下：
   <img src="/Images/back-to-top-chrome-extension/4.png"/>

# 调试用户脚本

如果需要调试我们写的脚本，可以在 Tempermonkey 的设置界面开启 Debug Scripts 功能。然后在打开页面的时候按 F12 开启 developer tools，就会自动进入调试模式了。如下图所示：
<img src="/Images/back-to-top-chrome-extension/5.png"/>  
<img src="/Images/back-to-top-chrome-extension/6.png"/>
