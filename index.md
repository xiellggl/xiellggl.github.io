# xiell个人博客
写个博客纯属是交流一下
## spring微服务实战
主要是说按照书上没有跑通，我自己的解决办法
父项目地址：https://github.com/xiellggl/mis-parent.git
说明：
1. 书上源码是同一个仓库，这里是用了submodule
2. 书上没有对docker和docker-compose的说明，而第九章后面是以这个为前提的
### 第九章：日志聚合中使用
### 第6章中：SpecialRoutesFilter中实际返回的是根据service id进行路由的结果，不是我们真正想要的结果，RibbonRoutingFilter始终会进行过滤，而覆盖掉你自己forward的结果，有两种解决方法
1. 使用静态路由，并设置context.setRouteHost(null);防止SimpleHostRoutingFilter对结果覆盖
2. 依旧使用service id进行路由，不过在进入到下一个route类型filter之前设置,然后在ResponseFilter中补回

下面给出第二种方法实现

在进入到下一个route类型filter之前设置
```diff
            Object serviceId = context.get("serviceId");
            context.remove("serviceId");
            context.put("copyServiceId", serviceId);          
``` 
在ResponseFilter中补回   
```diff                    
            if (ctx.get("serviceId") == null && ctx.get("copyServiceId") != null) {
            ctx.put("serviceId", ctx.get("copyServiceId"));
            ctx.remove("copyServiceId");
        }
```
如果以上方法不行，那么你可能还要把这句删掉，或者延迟执行
```diff
            httpClient.close();
```
### 第十章:部署微服务

终于到部署了，当我在兴致冲冲准备白嫖亚马逊，然后又终于能把东西扔上云，我也是用过云的人了，发现要信用卡才能注册，那么没有信用卡的怎么办呢？
大家可以到阿里云和腾讯云找免费的对应服务，一般在首页找找就会找到，找不到的时候你可以根据关键字`免费redis 阿里云`之类的去搜，我白嫖了阿里云和腾讯云硬是把书上的基础服务弄齐了。

#### jenkins

我没有跟书上一样，使用travis作为CI工具，原因是我觉得国内大多用jenkins

jenkins作为CI要注意的问题:

jenkins的插件下载慢，在线下如果不行的话，就设置代理，用upload phi文件方式的话太麻烦了，还要装几个依赖插件，而且我猜每个依赖插件都要clone插件项目自己build。。。
今天看了一下昨天白嫖的腾讯云的redis，是不给外网连接，也没什么所谓了，只是想连一下而已。
现在是半夜02:49分，终于把jenkins自动构建弄好了，弄了那么久
1. 自己的代理处理问题，还以为是jenkins怎么本地访问都这么慢
2. 处理jenkins时区问题，我按照官网的试了，jenkins日志还是差了8小时，官网解答地址`https://wiki.jenkins.io/display/JENKINS/Change+time+zone`
3. 配置github的webhooks的payload url，配了好久，最后根据知乎这篇`https://zhuanlan.zhihu.com/p/34758963`配好了，注意当你选择呢Trigger builds remotely时，注意看下面的文字，他有告诉你url是什么，我就是一开始没注意以为不是什么重要的东西，后来才发现，然后TOKEN_NAME就是你上面的AUTHENTICATION TOKEN，到这里还并没有弄好，postman直接访问是可以的，但是github还是不行，啊啊啊！
4. 现在是半夜5点，问题3找到根本原因了，我payload url填的是127.0.0.1啊，我的发，github怎么知道127.0.0.1是什么鬼！
两种解决办法：

            1. 用ngrok暴露本地ip到公网，他会返回像`http://3b2db437.ngrok.io`的url给你，填入到github中
            
            2. 上云自己弄一个服务器当jenkins服务器

本来打算白嫖腾讯云的服务器当jenkins的，谁知道刷新几次，就不给我用了，应该是怕我恶意白嫖吧，没办法我最后又找到了华为云
安装jenkins的时候yum在线安装太慢了，打算yum本地安装，谁知道ssh一直连不上去，最后发现是一开始输错几次，直接ip被拒绝了，可以在`/etc/hosts.deny`里面查看,如果出现这种情况，直接把自己ip的那条记录删掉就行了。

删掉之后就可以用scp上传的云服务器，
jenkins默认是创建一个jenkins用户启动服务的
太坑了！！！centos yum装的是1.7.1版本的git，太旧了，jenkins配置git的时候一直出现403 permission deny，我靠，太坑了，虽然一开始有查到可能是git版本的问题，可是这毕竟是yum上面最新的了，并且我换过yun源之后都是这个版本，心里感觉没什么问题！！！直到最后我仔细的看了git官网下面这段话！！！

```diff
Red Hat Enterprise Linux, Oracle Linux, CentOS, Scientific Linux, et al.
- RHEL and derivatives typically ship older versions of git. 
You can download a tarball and build from source, or use a 3rd-party repository such as the IUS Community Project to obtain a more recent version of git.
```

红色部分意思就是通常是旧的版本

##### 自己构建jenkins镜像
因为现在是白嫖的，十有八九我是不会续费的，那我肯定还是要继续研究微服务的，所以决定弄个镜像，就不用到时再配置
我晚点会把配置发上来，好像插件还是要自己装哎，最后决定使用镜像，然后挂载需要的东西，比如maven和git

然后将freestyle项目改用maven项目，需要下一个maven integration插件，然后还要下maven，建议使用自动安装。在使用jenkins镜像的情况下，自己安装还要挂载目录。

接下来是设置子模块自动maven和docker构建，目前只能是指定某一子模块进行构建，我也没看到，递归构建子模块的选项
　　解决办法：当你指定用父项目进行构建时，需要在其pom文件中把packaging改为pom，如果你是需要每个子模块都执行mvn命令的话，注意不要选择并行构建，并行构建只会构建父模块，不会把你的build指定具体到每个模块执行。
    
jenkins默认用户是uid=1000，你也可以在启动容器时使用-u更改，当显示
```
I/O exception (java.io.IOException) caught when processing request to {}->unix://localhost:80: Permission denied
```
这个错误时，表明jenkins镜像对/var/run/docker.sock没有权限，可使用chown 1000:1000 /var/run/docker.sock解决

由于华为云白嫖的服务器快到期了，所以我把服务器的jenkins的home目录，复制了写来，然后把镜像推到了阿里云的registry服务，我一早就想到了这个问题，所以当初才会选择以容器方式来运行jenkins
