# 微信小程序中间人监听破解（MITM：Man in the middle）

作者：Bumprat

编制日期：2022 年 9 月 18 日

PC微信版本 3.7.6.44

## 概述

1. 此方法主要思路是中间人监听，man in the middle，简洁易懂，几乎不用写代码
1. 为了方便地实现 http/https 请求捕获，需要使用 pc 环境
1. pc 环境微信 3.7.6.44 版本，微信小程序可执行文件不是主程序文件 WeChat.exe，而是 WeChatAppEx.exe
1. 主程序文件 WeChat.exe 在登陆时支持修改 http 代理，但对 WeChatAppEx.exe 不生效
1. 使用 Proxifier 强制 WeChatAppEx.exe 走 https 代理
1. 使用 Burpe Suite 捕获 http2 数据

## 环境配置

### Proxifier 环境

1.  配置代理服务器设置

    添加->地址 `127.0.0.1`->端口 `8080`->协议 `HTTPS`

1.  编辑代理规则

    修改 Default 的动作 action 为直接 `Direct`，防止其他应用请求混入，新建->名称`微信小程序`->应用程序 `WeChatAppEx.exe`，动作选择上一步添加的代理默认名为 `Proxy HTTPS 127.0.0.1`，关闭窗口会运行于系统托盘

### Burpe Suite 环境

Burpe Suite 基于 java（自带 jre，需要管理员权限，可以开启高 DPI 修复）

1.  首次启动会自动建立一个 8080 端口的监听代理服务器，如果不正确，可以在 Proxy->Options->Proxy Listeners 中增加，用浏览器访问监听端口可以下载根证书，双击根证书安装到`“受信任的根证书颁发机构”`中去

1.  User options->Display->HTTP Message Display->Font 修改字体为宋体，防止中文乱码

1.  Proxy->HTTP history 中可以看到请求历史记录，类似于 Fiddler

1.  Target->Site map 中可以开看到请求历史记录，类似于 Charles

## 修改服务器响应（MITM attack 中间人攻击）

1.  手动方法（测试可用）

    Proxy->Intercept，待被测试应用准备好后，点击 Intercept is off 按钮开启 intercept，然后启动应用数据交互，系统每捕捉到一个请求都会中断，点击 Action->Do intercept->Response to this request，点击 Forward，响应前会再次触发中断，响应内容可以编辑

2.  自动方法（未测试）

    Proxy->Options->Match and Replace->Add，这里可以使用正则表达式，当请求或响应满足一定条件时，触发替换。
