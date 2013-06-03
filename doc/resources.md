## Resources


### 插件信息
###### 最新版本：1.2
###### 下载地址：[http://grails.org/plugin/resources](http://grails.org/plugin/resources)
###### 官方文档：[http://grails-plugins.github.io/grails-resources/](http://grails-plugins.github.io/grails-resources/)
###### 作者：Marc Palmer, Luke Daley
###### 协议：[Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0.txt)


### 功能介绍
用于HTML资源管理，提供声明和模块化的方式来依赖管理各种css、js、image资源，功能完全替代g.resource，目前一些流行的CSS、JS框架都已经集成并支持resources插件。

### 安装步骤
在Build.groovy 文件做如下配置：

plugins {
   runtime ':resources:1.2'
}


### 使用说明
配置文件ApplicationResources.groovy

''
modules = {
    core {
        dependsOn 'jquery, utils'
        defaultBundle 'ui'
        resource url:'/js/core.js', disposition: 'head'
        resource url:'/js/ui.js'
    }

    utils {
        dependsOn 'jquery'

        resource url:'/js/utils.js' 
    }

    forms {
        dependsOn 'core,utils'
        defaultBundle 'ui'

        resource url:'/css/forms.css'
        resource url:'/js/forms.js'
    }
}''
