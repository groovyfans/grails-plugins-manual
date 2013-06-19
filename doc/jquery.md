## jquery


### 插件信息
###### 最新版本：1.10.0
###### 下载地址：[http://grails.org/plugin/jquery](http://grails.org/plugin/jquery}
###### 官方文档：[http://grails.org/plugins/jquery](http://grails.org/plugins/jquery)
###### 作者：Sergey Nebolsin, Craig Jones, Marc Palmer, Finn Herpich
###### 协议：[Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0.txt)


### 功能介绍
提供与流行JS框架jQuery的集成。

### 安装步骤
在Build.groovy 文件做如下配置：

plugins {
   runtime ":jquery:1.10.0"
}


### 使用说明
在layout/main.gsp中添加

	<g:javascript library="jquery" plugin="jquery"/>
