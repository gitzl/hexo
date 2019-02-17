---
title: 从0到1开启jenkins插件开发之路
---

##### 前言

项目中有这样一个需求，CI部分用jenkins实现拉取代码、编译等，CD需要实现自动部署到公司的Pass平台中,需要CI和CD联动实现自动化，因此需定制开发部署到Pass平台的jenkins插件，本文用一个简单的Demo记录下jenkins插件开发流程，以扩展jenkins构建阶段为例进行介绍.

##### 主要内容包括

```
- 创建jenkins插件项目
- 项目源码分析
- jelly界面开发
- 插件运行、打包、离线安装

```

<!-- more -->

##### 环境准备

```
- JDK8               //安装jdk8
- mave3             //安装maven3以上的版本
- javac -version   //确认jdk版本
- mvn   -version  //确认maven版本

```

##### create a  jenkins Plugin

```
mvn -U archetype:generate -Dfilter=io.jenkins.archetypes:

```

```
$ mvn -U archetype:generate -Dfilter=io.jenkins.archetypes:
…
Choose archetype:
1: remote -> io.jenkins.archetypes:empty-plugin (Skeleton of a Jenkins plugin with a POM and an empty source tree.)
2: remote -> io.jenkins.archetypes:global-configuration-plugin (Skeleton of a Jenkins plugin with a POM and an example piece of global configuration.)
3: remote -> io.jenkins.archetypes:hello-world-plugin (Skeleton of a Jenkins plugin with a POM and an example build step.)
Choose a number or apply filter (format: [groupId:]artifactId, case sensitive contains): : 3  #选择hello-world-plugin
Choose io.jenkins.archetypes:hello-world-plugin version:
1: 1.1
2: 1.2
3: 1.3
Choose a number: 3: 3  #选择最新version 1.3 
[INFO] Using property: groupId = unused
Define value for property artifactId: demo # 取一个项目名
Define value for property version 1.0-SNAPSHOT: : #版本
[INFO] Using property: package = io.jenkins.plugins.sample
Confirm properties configuration:
groupId: unused
artifactId: demo
version: 1.0-SNAPSHOT
package: io.jenkins.plugins.sample
 Y: : y #确定创建plugin
```
##### make sure we can build plugin
```
$ mv demo demo-plugin  #重命名为：demo-plugin 
$ cd demo-plugin
$ mvn verify           # 验证是否能正常编译

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 06:11 min
# SUCCESS 说明创建的Plugin能正常的编译
```
关于hpi插件可参考: [https://jenkinsci.github.io/maven-hpi-plugin/](https://jenkinsci.github.io/maven-hpi-plugin/)

##### use maven hpi Plugin  
```
mvn hpi:run
# INFO: Jenkins is fully up and running 项目已正常启动
# http://localhost:8080/jenkins/ 访问项目
```


##### Demo项目源码分析

```
#Demo项目源码主要内容结构如下，下面将分析java以及resources包下的内容

Demo
  -java
    -io.jenkins.plugins.sample
       -HelloWorldBuilder.java
  -resources
    -io.jenkins.plugins.sample
      -HelloWorldBuilder
        -config.jelly
        -index.jelly
        -help-name.html
```

> HelloWorldBuilder.java源码分析

###### 扩展构建阶段

```
public class HelloWorldBuilder extends Builder implements SimpleBuildStep {
    private final String name;
}
#继承Builder扩展构建阶段

```
###### 数据绑定

```
- DataBoundConstructor

    @DataBoundConstructor
    public HelloWorldBuilder(String name) {
        this.name = name;
        
    }
# 构造函数初始化值，加上DataBoundConstructor注解进行数据绑定，对应jelly页面数据结构

```

###### perform方法

```
@Override
public void perform(Run<?, ?> run, FilePath workspace, Launcher launcher, TaskListener listener) throws InterruptedException, IOException {
            listener.getLogger().println("Hello, " + name + "!");
            //逻辑处理
}
    
#重写perform函数，运行时会执行perform方法，因此在perform方法中实现插件逻辑处理
#这里的逻辑是将界面传过来的name值写入到jenkins的log中，方便在运行日志中进行调试

```
###### check and displayName 

```
    @Symbol("greet")
    @Extension
    public static final class DescriptorImpl extends BuildStepDescriptor<Builder> {
        
        //检测界面填写的字段是否合法
        
        public FormValidation doCheckName(@QueryParameter String name)
                throws IOException, ServletException {
            
            if (name.length() == 0)
                return FormValidation.error("名称不能为空");
           
            return FormValidation.ok();
        }

        @Override
        public boolean isApplicable(Class<? extends AbstractProject> aClass) {
            return true;
        }
        
        //重写DisplayName方法，该名称会在界面的构建阶段下拉框中显示

        @Override
        public String getDisplayName() {
            return "Demo";
        }
    }
    
```


> resources内容分析

######  创建HelloWorldBuilder文件夹

```
在/resource/io.jenkins.plugins.sample包创建HelloWorldBuilder文件夹，与HelloWorldBuilder.java的类名对应，界面的数据字段才能通过HelloWorldBuilder.java构造函数进行绑定

```
###### jelly界面开发

```

#界面开发主要包括以下模块

1. config.jelly             #插件主页面
2. help-name.html           #jenkins界面点击问号出现的提示内容，命名规则:help-字段名.html
3. index.jelly              #整体布局，扩展构建阶段不涉及index.jelly

```

###### config.jelly

```
<?jelly escape-by-default='true'?>
<j:jelly xmlns:j="jelly:core" xmlns:st="jelly:stapler" xmlns:d="jelly:define" xmlns:l="/lib/layout" xmlns:t="/lib/hudson" xmlns:f="/lib/form">
    <f:entry title="名称" field="name">
        <f:textbox />
    </f:entry>
</j:jelly>

#这里是一个文本输入框，字段为name，name和HelloBuild.java构造函数传入的name对应，在HelloBuilder.java中即可获取到界面传过来的值

```
jelly语法参考：http://commons.apache.org/proper/commons-jelly/tutorial.html

##### 打包 & 离线部署

```
- 运行  mvn hpi:run 
- 打包  mvn package or mvn clean package -Dmaven.test.skip=true #生成插件的路径在target/demo.hpi
- 部署  jenkins-系统管理-插件管理-高级  上传demo.hpi插件，重启jenkins

```

##### Demo效果展示

构建下拉框增加Demo选项

![image](./img/jenkins/jp1.png)

设置名称为：demo

![image](./img/jenkins/jp2.png)

执行日志

![image](./img/jenkins/jp3.png)

##### 参考文档
- jenkins插件入门：https://jenkins.io/doc/developer/tutorial/
- jenkins开发文档：https://jenkins.io/doc/developer/book/
- jenkins扩展点：https://jenkins.io/doc/developer/extensions/
- maven hpi插件：https://jenkinsci.github.io/maven-hpi-plugin/
- jelly：http://commons.apache.org/proper/commons-jelly/tutorial.html
