#  漏洞预警 | 泛微e-cology9 SQL注入漏洞  
浅安  浅安安全   2025-07-15 00:00  
  
**0x00 漏洞编号**  
- QVD-2025-26680  
  
**0x01 危险等级**  
- 高危  
  
**0x02 漏洞概述**  
  
泛微协同管理应用平台e-cology是一套兼具企业信息门户、知识文档管理、工作流程管理、人力资源管理、客户关系管理、项目管理、财务管理、资产管理、供应链管理、数据中心功能的企业大型协同管理平台。  
  
![图片](https://mmbiz.qpic.cn/mmbiz_png/7stTqD182SWNxyZy7deUhOtianY0EZZdVdL5WcWVjIg2icvfrGibKs3Dd0FeTmLeIfYF0FAh2YCQ72IWauRGohmsA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&tp=webp "")  
  
**0x03 漏洞详情**  
  
QVD-2025-26680  
  
**漏洞类型：**  
SQL注入  
  
**影响：**  
敏感信息泄露  
  
**简述：**  
泛微  
E-cology9存在SQL注入漏洞，由于对用户传入的参数没有严格校验，攻击者可利用该漏洞获取数据库敏感信息，进而获取服务器权限。  
  
**0x04 影响版本**  
- 泛微E-cology9  
  
**0x05 POC状态**  
- 未公开（已有复现）  
  
**0x06 修复建议**  
  
**目前官方已发布漏洞修复版本，建议用户升级到安全版本**  
**：**  
  
https://www.weaver.com.cn/cs/securityDownload.asp  
  
  
  
