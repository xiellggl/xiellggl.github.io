## xiell个人博客
写个博客纯属是交流一下
### spring微服务实战
主要是说按照书上没有跑通，我自己的解决办法
父项目地址：https://github.com/xiellggl/mis-parent.git
说明：
1.书上源码是同一个仓库，这里是用了submodule
2.书上没有对docker和docker-compose的说明，而第九章后面是以这个为前提的
#### 第九章：日志聚合中使用
#### 第6章中：SpecialRoutesFilter中实际返回的是根据service id进行路由的结果，不是我们真正想要的结果，RibbonRoutingFilter始终会进行过滤，而覆盖掉你自己forward的结果，有两种解决方法
1. 使用静态路由，并设置context.setRouteHost(null);防止SimpleHostRoutingFilter对结果覆盖
2. 依旧使用service id进行路由，不过在进入到下一个route类型filter之前设置
```
            Object serviceId = context.get("serviceId");
            context.remove("serviceId");
            context.put("copyServiceId", serviceId);
然后在ResponseFilter中补回
 if (ctx.get("serviceId") == null && ctx.get("copyServiceId") != null) {
            ctx.put("serviceId", ctx.get("copyServiceId"));
            ctx.remove("copyServiceId");
        }
如果以上方法不行，那么你可能还要把这句删掉，或者延迟执行
httpClient.close();
```
#### 第十章:部署微服务
终于到部署了，当我在兴致冲冲准备白嫖亚马逊，然后又终于能把东西扔上云，我也是用过云的人了，发现要信用卡才能注册，那么没有信用卡的怎么办呢？
大家可以到阿里云和腾讯云找免费的对应服务，一般在首页找找就会找到，找不到的时候你可以根据关键字"免费redis 阿里云"之类的去搜，我白嫖了阿里云和腾讯云硬是把书上的基础服务弄齐了。
