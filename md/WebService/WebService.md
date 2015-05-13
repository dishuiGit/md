[TOC]

## WebService注解的使用

#### 服务端
1. 创建一个天气对象的实体.包括城市,日期,天气,温度
2. 创建一个SEI
3. 创建SEI的实现类,加上`@WebService`注解
4. 发布`webservice`. `Endpoint.publish`

#### 客户端
1. 基于wsdl生成客户端调用代码
2. 创建服务视图
3. 获得porttype
4. 调用服务端方法

--------------------------------------------

### 服务端
`WeatherInterface.java`

```java
/**
 * 天气查询服务端
 */
public interface WeatherInterface {

    List<WeatherModel> queryWeatehr(String cityname);
    
}
```

`WeatherInterfaceImpl.java`

```java
@WebService(
        serviceName="WeatherService"        //服务视图的名字
        ,portName="WeatherPort"             //port节点的name属性
        ,name="WeatherInterface"            //PortType节点的name属性
        ,targetNamespace="http://weather.itcast.cn/"    //命名空间的名称
)
public class WeatherInterfaceImpl implements WeatherInterface {

    @Override
//  @WebResult(name="WeatherModelList")
    @WebResult(name="三天的天气信息")
    public List<WeatherModel> queryWeatehr(@WebParam(name="cityname")String cityname) {
        List<WeatherModel> resultList = getWeather(cityname);
        
        return resultList;
    }
    
    @WebMethod(exclude=true)
    public List<WeatherModel> getWeather(String cityName) {
        List<WeatherModel> resultList = new ArrayList<>();
        Calendar calendar = Calendar.getInstance();
        //第一天
        WeatherModel model1 = new WeatherModel();
        model1.setCityname(cityName);
        model1.setDate(calendar.getTime());
        model1.setInfo("今天天气不错");
        model1.setMaxTemp(22);
        model1.setMinTemp(10);
        resultList.add(model1);
        //第二天
        WeatherModel model2 = new WeatherModel();
        model2.setCityname(cityName);
        calendar.set(Calendar.DATE, calendar.get(Calendar.DATE) + 1);
        model2.setDate(calendar.getTime());
        model2.setInfo("今天天气不错");
        model2.setMaxTemp(22);
        model2.setMinTemp(10);
        resultList.add(model2);
        //第三天
        WeatherModel model3 = new WeatherModel();
        model3.setCityname(cityName);
        calendar.set(Calendar.DATE, calendar.get(Calendar.DATE) + 1);
        model3.setDate(calendar.getTime());
        model3.setInfo("今天天气不错");
        model3.setMaxTemp(22);
        model3.setMinTemp(10);
        resultList.add(model3);
        
        return resultList;
    }

}
```

### 客户端
`WeatherClient.class`

```java
/**
 * 客户端调用程序
 */
public class WeatherClient {

    public static void main(String[] args) throws Exception {
        //创建服务视图
        Service service = Service.create(new URL("http://127.0.0.1:12345/weather"), 
                new QName("http://service.weather.itcast.cn/", "WeatherInterfaceImplService"));
        //获得porttype
        WeatherInterfaceImpl portType = service.getPort(WeatherInterfaceImpl.class);
        //调用服务端方法
        List<WeatherModel> list = portType.queryWeatehr("北京");
        //打印列表
        for (WeatherModel model:list) {
            System.out.println(model.getCityname());
            System.out.println(model.getInfo());
            System.out.println(model.getMaxTemp());
            System.out.println(model.getMinTemp());
            System.out.println(model.getDate().toGregorianCalendar().getTime().toLocaleString());
        }
    }
}
```
























