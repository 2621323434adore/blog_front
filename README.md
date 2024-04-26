# blog
## 项目构成
该项目采用前后端分离技术开发
## 项目技术
### 前端：

- vue
- elementUI
- pubsub-js
- Animate.css
- vue-router
- tinyMCE
- axios

### 后端：

- spring boot
- Mybatis-plus
- RESTful 风格
- sa-token
- hutools
- avtiviti

### 数据库：
MySQL

### 准备工作
#### 1、安装ElementUI
```shell
npm i element-ui -S
```
#### 2、安装pubsub-js
```shell
npm i pubsub-js
```
#### 2、安装Animate.css
```shell
npm install animate.css --save
```
#### 2、安装vue-router
```shell
npm install vue-router@3
```
#### 2、安装tinyMCE
```shell
npm install tinymce
```
#### 2、安装axios
```shell
npm install axios
```
<br>

# 软件详情
## 导航栏设置
### 首页

#### 功能点：

* 





### 文章专区
### 消息中心
### 社区


## 功能点设定
#### 登录与注册
#### 文章的发布
#### 文章的展示
#### 文章的的热度
#### 评论功能
#### 点赞功能
#### 文章专区
#### 文章搜索

<br>

### 角色设定
| 角色       | 功能                               | 是否正常使用 |      |
| ---------- | ---------------------------------- | ------------ | ---- |
| 游客角色   | 网站首页                           | 正常使用     |      |
|            | 文章阅读                           | 正常使用     |      |
|            | 文章评论                           | 无法正常使用 |      |
|            | 热点文章                           | 无法查看     |      |
|            | 文章发布                           | 无法使用     |      |
|            |                                    |              |      |
| 普通用户   | 网站首页                           | 正常使用     |      |
|            | 文章阅读                           | 正常使用     |      |
|            | 文章评论                           | 正常使用     |      |
|            | 热点文章                           | 正常使用     |      |
|            | 文章发布                           | 正常使用     |      |
|            | 个性头像，个性名称，称号显示       |              |      |
|            |                                    |              |      |
|            |                                    |              |      |
| 专区管理员 | 除基础功能、额外权限功能、个性展示 | 正常使用     |      |
|            | 文章审核                           | 正常使用     |      |
|            | 文章评论控制                       | 正常使用     |      |
|            | 小黑屋管理                         | 正常使用     |      |
|            | 个性头像，个性名称，称号显示       | 正常使用     |      |
|            | 专区设置（包括设置3个副管理）      | 正常使用     |      |
|            |                                    |              |        |
|            |                                    |              |      |
| 系统管理员 | 拥有所有权限                       | 正常使用     |      |

<br>

### 审核机制

#### 文章审核

为了社区的友好与社区的和平，我们需要对文章进行审核，审核不通过的将无法将文章正常发布

#### 文章评论审核（待议）

#### 文章热度榜审核

为了防止恶意刷榜，热度大于五百的文章在上榜前需要经由专区管理或系统管理员审核通过可上榜

#### 用户权限升级审核

用户可以申请建立专区，在专区创建审核通过后该用户将成为该专区的专区管理员

## 开发时遇到的问题
### 1、使用spring boot + sa-token 做认证登录时我发现前端与后端存在跨域问题
```JavaScript
// 前端的解决思路：
/*
* 前端配置中设置了proxy，将所有以/api开头的请求代理到http://127.0.0.1:5700上，
* 但是在实际请求时，您的请求路径为http://127.0.0.1:5700/api/test。
* 这会导致浏览器发出跨域请求，从而触发浏览器的同源策略，导致跨域错误。为了解决这个问题，
* 您可以将前端请求路径设置为http://127.0.0.1:5700/api/login，以保持与后端请求路径的一致性。
* 同时，在后端代码中添加@CrossOrigin注解，允许跨域请求。
* */
//配置
devServer:{
    proxy:{
        '/api': {
            target: 'http://127.0.0.1:5700',//后端接口地址
                changeOrigin: true,//是否允许跨域
                pathRewrite: {
                '^/api': '/'//重写,
            }
        }
    }
}
```
```javascript
// 请求实例
gotoLogin(){
    axios({
        url:"/api/login",
        method:"post",
        data:{
            username:this.username,
            password:this.password
        }
    }).then((res)=>{
        console.log(res)
    })
}
```
```java 
@CrossOrigin("*")
@RestController
@RequestMapping("/api")
public class UserController {

  @Autowired
  UserServiceImpl userServiceImpl;

  @PostMapping("/login")
  public SaResult Login(@RequestParam String username, @RequestParam String password, HttpServletResponse response){
    response.setHeader("Access-Control-Allow-Origin", "*");
    response.setHeader("Access-Control-Allow-Methods", "POST");
    String encode = PasswordUtil.encode(password);
    XiRangUser login = userServiceImpl.login(username, encode);
    if(login != null){
      StpUtil.login(login.getId());
      return SaResult.ok().set("UserInfo",login).set("tokenInfo", StpUtil.getTokenInfo());
    }else{
      return SaResult.error("登录失败！");
    }
  }
}

```
### 2、在解决跨域问题后 发现Vue 提交的数据后端始终无法获取
原因：在 HTTP 协议中，如果前端使用 POST 方法提交 JSON 格式的数据，那么数据实际上是以请求体的形式传递的，
而不是像 GET 方法那样以查询字符串的形式传递。
而 Spring MVC 在默认情况下是不支持以请求体的形式接收参数的，需要手动开启。
```properties
spring.mvc.converters.preferred-json-mapper=jackson
spring.http.encoding.force=true
spring.http.encoding.charset=UTF-8
spring.http.encoding.enabled=true
# preferred-json-mapper 参数指定了使用 Jackson 来转换 JSON 数据，
# encoding 相关的参数用于设置字符编码，force 参数用于强制开启编码。
# 如果你使用的是其它的 JSON 转换库，可以将上面的 jackson 替换成你使用的库的名称。
```
这个问题也可以在前端使用formdata 来解决
### 3、文章展示时出现排列不整齐问题
原因：通过调试工具发现是文章内容直接贴到页面上导致容器撑大造成的
解决方式：
```css
.article{
  margin: 5px 5px;
  height: 215px;
/**给父容器设置一个固定高度*/
}
```
### 4、文章内容在首页预览展示时出现页面混乱不一致
原因：
1、因为是之前版本使用的是html标记直接记录入数据库的在做文章预览时如果直接做html插入文章中各种html标签将会被浏览器渲染从而导致页面混乱  
2、如果使用文本的方式插入到页面，页面不会造成混乱，但是html的标签也将会作为文章内容进行预览，页面文字将会过多导致用户体验不佳  
解决方法：  
一、思路：使用工具将 HTML 转换为 markdown 然后再展示到前端页面上  
1、后端导入pom
```xml
     <dependency>
            <groupId>com.kotcrab.remark</groupId>
            <artifactId>remark</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>com.github.houbb</groupId>
            <artifactId>html2md</artifactId>
            <version>0.0.1</version>
        </dependency>
```
这两个插件是用于将html转换到markdown的  
2、在测试类中直接对数据进行修改就可以了(修改数据前记得`备份` `备份` `备份`)
因为这里是直接对做操作一定要备份
```java
 @Test
    public void ArticleUpDateTextToMarkdown() throws IOException {
        List<Article> list = articleService.list();
        Remark remark = new Remark();
        for(Article article : list){
            String convert = remark.convert(article.getText());
            article.setText(convert);
            System.out.println(articleService.UpdateArticle(article));
        }
    }
```

### 5、前端设置了HTTP请求头后端接收不到HTTP请求头，并报出空指针异常
原因：因为该请求属于跨域请求所以该请求的请求头将会被丢弃
解决方式：
```java
@CrossOrigin  // 为请求路径添加跨域注解
```

