
### 品牌分类
#### 根据条件查询所有品牌
+ 分析页面传递的参数
    + 查询条件(名称,是否可见)
    + 当前页数


### 遇到的问题 :
+ Tomcat __get请求乱码__
    + `$TOMCAT_HOME/conf/server.xml` 
        +  `<Connector URIEncoding="UTF-8" connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>`
+ 添加分页显示对象 pageView  ==> /brand/list.do?name=依琦莲&isDisplay=1&pageNo=2
+ Tomcat图片服务器(可读可写)
    + 修改 `$TOMCAT_HOME/conf/web.xml`
``` xml
<servlet>
        <servlet-name>default</servlet-name>
        <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
        <init-param>
            <param-name>debug</param-name>
            <param-value>0</param-value>
        </init-param>
        ------修改------
        <init-param>
            <param-name>readonly</param-name>
            <param-value>false</param-value>
        </init-param>
        ---------------
        <init-param>
            <param-name>listings</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
```
### 全选/批量删除
>如果点击事件提交form为成功,首先检查是否form表单存在

``` javascript
<script type="text/javascript">
//全选
function checkBox(name,checked){
    $("input[name='"+name+"']").each(function(){
        $(this).attr("checked",checked);
    })
}
//批量删除(用表单批量删除)
function optDelete(name,isDisplay,pageNo){
    if($("input[name='ids']:checked").size() == 0){
        alert("请至少选择一个");
        return;
    }
    if(!confirm("你确定删除吗?")){
        return;
    }
    //  /brand/brands_Delete.do?
    //  <c:if test='${param.name!=null }'>name=" + name + "&</c:if>
    //  <c:if test='${param.isDisplay!=null }'>isDisplay=" + isDisplay + "&</c:if>
    //  pageNo=" + pageNo

    var url = "/brand/brands_Delete.do?<c:if test='${param.name!=null }'>name=" + name + "&</c:if><c:if test='${param.isDisplay!=null }'>isDisplay=" + isDisplay + "&</c:if>pageNo=" + pageNo;
    //js提交Form表单                         //提交  指定GET  POST
    alert(url);
    $("#jvForm").attr("action",url).attr("method","post").submit();
}
</script>
```


### 上传图片

```java
public class UploadController {

    //上传图片
    @RequestMapping("/upload/uploadPic")
    public void uploadPic(@RequestParam MultipartFile pic, HttpServletResponse response){
        //扩展名
        String ext = FilenameUtils.getExtension(pic.getOriginalFilename());
        //图片名称生成策略
        DateFormat df = new SimpleDateFormat("yyyyMMddHHmmssSSS");
        String name = df.format(new Date());
        //随机三位数,生成图片名称
        Random r = new Random();
        for(int i=0;i<3;i++){
            name +=r.nextInt(10);
        }
        //使Jersey框架
        Client client = new Client();
        String path = "upload/" + name +"."+ext;
        //public static final String IMG_WEB = "http://localhost:8088/img-web/";
        String url = Constants.IMG_WEB + path;
        //发送图片
        WebResource resource = client.resource(url);
        try {
            resource.put(String.class, pic.getBytes());
        } catch (IOException e) {
            e.printStackTrace();
        }
        //返回路径
        JSONObject jo = new JSONObject();
        jo.put("url", url);
        jo.put("path", path);
        //告诉浏览器,你发送的数据类型
        response.setContentType("application/json;charset=UTF-8");
        try {
            response.getWriter().write(jo.toString());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### SprintMVC拦截器,上传图片

```xml
    <mvc:annotation-driven conversion-service="conversionService" />

    <bean id="conversionService"
        class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <!-- 日期转换 -->
                <bean class="cn.dishui.common.convertion.CustomDateConverter" />
                <!-- 参数 传递 前后去掉空格 -->
                <bean class="cn.dishui.common.convertion.CustomTrimConverter" />
            </list>
        </property>
    </bean>

        <!-- 配置Springmvc支持上传图片 -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 指定上传大小  上传的图片不能超过1M    默认是 B -->
        <property name="maxUploadSize" value="1048576"/>
    </bean>

```

### 添加图片,异步提交

``` javascript
function uploadPic(){
    
    //form 给你样本  jvForm
    //请求路径 上传哪里
    var options = {
            url : "/upload/uploadPic.do",
            type : "post",
            dataType : "json",
            success : function(data){
                //返回二个路径
                //图片标签 src
                $("#allImgUrl").attr("src",data.url);
                $("#path").val(data.path);
            }
    }
    //异步提交
    $("#jvForm").ajaxSubmit(options);
    
}
```
