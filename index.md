## xiell个人博客
写个博客纯属是交流一下

### spring微服务实战
主要是说按照书上没有跑通，我自己的解决办法
父项目地址：https://github.com/xiellggl/mis-parent.git
说明：
1.书上源码是同一个仓库，这里是用了submodule
2.书上没有对docker和docker-compose的说明，而第九章后面是以这个为前提的
第九章：日志聚合中使用
第6章中：SpecialRoutesFilter中实际返回的是根据service id进行路由的结果，不是我们真正想要的结果，RibbonRoutingFilter始终会进行过滤，而覆盖掉你自己forward的结果，有两种解决方法
1.使用静态路由，并设置context.setRouteHost(null);防止SimpleHostRoutingFilter对结果覆盖
2.依旧使用service id进行路由，不过在进入到下一个route类型filter之前设置
```markdown
            Object serviceId = context.get("serviceId");
            context.remove("serviceId");
            context.put("copyServiceId", serviceId);
然后在ResponseFilter中补回
 if (ctx.get("serviceId") == null && ctx.get("copyServiceId") != null) {
            ctx.put("serviceId", ctx.get("copyServiceId"));
            ctx.remove("copyServiceId");
        }
  
```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/xiellggl/xiellggl.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
