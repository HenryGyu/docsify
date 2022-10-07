# Skywalking自定义链路追踪

默认情况下，Skywalking的追踪列表仅显示了controller层的路径url，没有显示service层的业务方法。  
如需要对业务中的方法，实现链路追踪，可以按照下面步骤实现。

**1、Maven添加依赖**

```xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-trace</artifactId>
    <!-- 版本和agent版本保持一致 -->
    <version>8.12.0</version>
</dependency>
```

**2、在业务方法上加入@Trace注解即可**

```java
@Slf4j
@Service
public class SysNoticeService extends ServiceImpl<CommSysNoticeDao, SysNoticeDO>
  implements IService<SysNoticeDO> {

  @Trace
  @Tags({
    @Tag(key = "param", value = "arg[0]"),
    @Tag(key = "result", value = "returnedObj")
  })
  public SysNoticeVO getById(Long id) {
    return SysNoticeConverter.INSTANCE.do2Vo(super.getById(id));
  }
}
```

> 注意：
> * 在方法上添加@Trace注解即可。如果想要看方法的入参和出参，才需要用到@tag或@tags注解。
> * 如果要用@tag或@tags注解，前提是必须要使用@Trace注解，不然仅仅给业务方法加@Tag注解的话，SkyWalking也不会显示。
> * @Tag注解中key我们可以自定义，而value的写法就固定了，如果要查看返回值就只能写returnedObj，如果要查看请求参数就只能用arg，下标代表第几个请求参数
> * 返回值的对象，注意要重写toString()方法，不然在SkyWalking的界面中显示的只是一个对象的内存地址

**3、在Skywalking-UI页面查看**

_![图片](../_media/Snipaste_2022-10-07_19-25-22.png ':size=100%')

_![图片](../_media/Snipaste_2022-10-07_19-29-58.png ':size=100%')
