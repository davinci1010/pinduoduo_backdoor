V2

# 对拼多多app利用0day漏洞控制用户手机及窃取数据的分析，含分析指引

## 结论： 
拼多多app中“内置”了用于攻击安卓手机厂商自带应用的Bundle风水漏洞（含OPPO、VIVO、华为、小米、三星等手机厂商），获取系统级StartAnyWhere 能力，从而完整控制用户手机，用于定向获取用户数据、自动启动、防卸载等。以上代码，任何了解安卓技术的技术人员，在拼多多apk中都能分析到相关行为，甚至调试复现漏洞，详见下文。

### 关键代码及分析概要：
- 关键代码：下文证明材料中，含拼多多apk文件中内置的0day漏洞代码，以及利用漏洞 StartAnyWhere 后，获取数据控制手机的代码。  

- 关键技术点分析：拼多多如何利用StartAnyWhere 能力，StartAnyWhere 的利用要相对复杂，它需要利用一些高权限的应用（如手机系统应用）自身所具备的功能来实现自己的目的，一般很难做到直接返回高权限shell。这也是为什么，网传拼多多招聘了一百多人的“手机厂商定向优化团队”，拼多多需要研究不同手机的高权限应用的功能，用它们的权限和功能实现自己控制用户手机的目的。  
- 需要说明，系统应用正常是不能被其他APP这样操作的，拼多多利用了0day漏洞实现 StartAnyWhere ，从而控制了系统应用。

### 吐个槽：
- 拼多多这样做在商业上能获得什么：非专业的一些解释，例如你在视频网站上刷视频，视频网站就知道你在这里的喜好，其他公司想服务这部分用户群体就要花钱找这个网站的公司投广告，这也是全球互联网商业的主要模式，拼多多的玩法是对这个模式的降维打击，自己可以拿到最全的数据，可以控制用户的流量，一年省出几百亿广告费用？这可能无法用钱评估了，很多东西，有钱也买不到。

- 能看懂这篇文章的人都能看出来，拼多多app做了什么，都刻在互联网的记忆中，瞒不住也辩解不了，现在Google应用保护和国外的杀毒软件也都将拼多多判定为恶意软件。拼多多真是丢了全中国的脸儿，网上看到很多人说拼多多是这样，觉得中国的app都会这样做？真给大伙儿丢人。



## 证明:

### 一、背景&用于分析的基础材料 
拼多多app近几年的版本都有问题，6.10之后的版本尤其嚣张，直接将漏洞利用代码打包到apk中，最新的6.50版因被曝光利用漏洞，删除了apk包中的漏洞利用代码，建议用6.10-6.49之间的版本做分析。 
背景，和启动分析的材料，直接看下面的链接，没有安卓逆向基础，直接下载《拼多多脱壳后的部分代码》，把文件夹拖到jadx里，看拼多多的java代码。 

- 《深蓝洞察 2022年度最“不可赦”漏洞》 https://mp.weixin.qq.com/s/P_EYQxOEupqdU0BJMRqWsw 
- 《拼多多apk内嵌提权代码，及动态下发dex分析》https://github.com/davinci1010/pinduoduo_backdoor 
- 《拼多多VMP脱壳机》https://github.com/davinci1012/pinduoduo_backdoor_unpacker 
- 《拼多多脱壳后的部分代码》https://github.com/poorjobless/pinduoduo_backdoor_code 
- 《拼多多利用漏洞攻击用户手机材料汇总&存证》https://github.com/recorder1013/pinduoduo_backdoor_recorder 


结论和上文一致，下文重点指出0day代码的位置，以及获取数据的代码的位置。  

	参考国外的分析结果，Google判定拼多多app为恶意应用，在应用市场下架了拼多多，杀毒软件对拼多多apk报毒。 


### 二、拼多多apk中恶意代码的位置

参考材料：《Bundle风水-Android parcel序列化和反序列化不匹配漏洞》https://i.blackhat.com/EU-22/Wednesday-Briefings/EU-22-Ke-Android-Parcels-Introducing-Android-Safer-Parcel.pdf 

拼多多apk中相关的0day 漏洞利用代码，如下图，有华为、小米、OPPO、VIVO、三星、以及通用设备的漏洞利用代码，目前大多还能用。 
自己要分析的话，下载上文的《拼多多脱壳后的部分代码》的GitHub，找到old_alive_base_ability_with_symbol目录，拖到jadx里，根据下图的路径就能看到。 

<img width="1303" alt="提权" src="https://user-images.githubusercontent.com/128487541/226639024-572e93d0-3d42-455f-8f3b-802e444f77e9.png">


通过上述的漏洞实现 StartAnyWhere 能力后（该能力如何利用的分析见下文第三章），利用该能力控制用户手机。用于获取用户隐私数据，控制用户手机的代码很多（此处不包含此前有大佬发过的动态下发的dex部分），有获取微博、bilibili类的社交媒体账号的模块，有获取用户各类搜索记录的模块，有获取用户手机通知、手机app使用记录、手机网络和定位等信息的模块，甚至还上传了手机语音助手（如小爱同学）的日志文件，也不知道里面记录了啥，东西太多。 
下文给出代码位置，可自行分析，重点关注dataCollect、idCollect路径下的代码。 

上文的《拼多多脱壳后的部分代码》的GitHub，找到alive_strategy_biz_plugin目录，拖到jadx里，根据下图的路径就能看到。  
<img width="1276" alt="alive_strategy_biz_plugin" src="https://user-images.githubusercontent.com/128487541/226658238-bb6c36f8-62c3-4dce-a43b-b9bd8f7c3a93.png">


上文的《拼多多脱壳后的部分代码》的GitHub，找到smart_shortcut_plugin目录，拖到jadx里，根据下图的路径就能看到。  
<img width="1304" alt="smart_shortcut_plugin" src="https://user-images.githubusercontent.com/128487541/226658230-f84853fe-4248-4a29-8231-5fa8ab70cc1e.png">



### 三、关键技术点分析-拼多多如何利用 StartAnyWhere 能力：
StartAnyWhere 能力简单理解，就是可以绕过安卓系统的对app的权限验证，调任意app内的功能，比如可以不知道手机的密码，直接打开设置中修改密码的页面。
拼多多在对 StartAnyWhere 实际的利用中，需要研究不同手机的高权限应用中可以通过 StartAnyWhere 调用的功能，用它们的权限和功能实现自己控制用户手机的目的，比如读写文件、安装卸载应用、修改应用权限等，这些功能组合起来还能实现对手机的持久化控制。 

通过拼多多获取微博账号的代码举例： 
代码中读取了文件“/data/system_ce/0/accounts_ce.db”该文件是系统文件，从中查询了微博账号，如下图
``` java
 cursor = sQLiteDatabase.query("accounts", null, "type = ?", new String[]{"com.sina.weibo.account"}, null, null, null);
``` 
<img width="1379" alt="StartAnyWhere1" src="https://user-images.githubusercontent.com/128487541/226679200-9871c86b-acb9-4bf9-82a1-270297ba9c4a.png">

拼多多app本身无权限读取该文件，但通过 StartAnyWhere 能力绕过系统权限控制，利用系统应用中可以读取文件的“接口”，来实现读取其他应用文件的目的，如下图： 

<img width="1376" alt="StartAnyWhere2" src="https://user-images.githubusercontent.com/128487541/226679189-327a228c-820e-4740-bd4b-4b6247eed53a.png">


上图中，拼多多app为了能读取文件，就要针对不同厂商手机进行”适配“（挖洞？），工作量巨大，听说招了一百多人来做“手机厂商定向优化“，“非常辛苦”。他们要借助众多系统应用实现 读写文件、安装卸载应用、修改应用权限 等一系列功能。  
通过以上能力，拼多多针对有持续控制需求的设备，通过替换系统apk或替换系统apk的动态链接库文件，取得持久化的system权限后门，这些在代码中也都有体现，可自行分析。



