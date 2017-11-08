---
layout: post
keywords: motan,restful,exception
description:  motan框架restful协议统一异常处理
title: "motan框架restful协议统一异常处理"
categories: [rpc]
tags: [motan, rpc]
group: archive
icon: file-alt
---
{% include codepiano/setup %}

### 通过自定义 EnpointFactory 处理 restful 协议的业务异常

motan 没有暴露 resteasy 提供的 @Provider 机制来处理异常，如果想利用这种机制，需要自己实现一个 EndpointFactory 来替换 motan 中自带的，插入自己实现的 @Provider 类

#### 一个简单实现

1. 新建一个 maven 工程，添加 motan restful 协议的 jar 包依赖到 pom 文件，注意 scope 为 provided

    ```
    <dependency>
        <groupId>com.weibo</groupId>
        <artifactId>motan-protocol-restful</artifactId>
        <version>0.3.1</version>
        <scope>provided</scope>
    </dependency>
    ```
1. 新建一个用来统一处理异常的类，例如`ExampleExceptionMapper`，实现可以参照`com.weibo.api.motan.protocol.restful.support.RpcExceptionMapper`，注意接口`ExceptionMapper<Exception>`支持泛型
1. 新建一个`EndpointFactory`实现类，例如`ExampleEndpointFactory`，继承`com.weibo.api.motan.protocol.restful.support.AbstractEndpointFactory`类，在类上添加`@SpiMeta`注解，指定 name 属性值，例如 'example'。
1. 实现`AbstractEndpointFactory`类中的`protected abstract RestServer innerCreateServer(URL url);`方法，大部分代码可以照搬`com.weibo.api.motan.protocol.restful.support.netty.NettyEndpointFactory` 类中的实现，需要修改下面的一行代码，其余可以保持不变

    ```
        # 把 RpcExceptionMapper.class.getName() 替换为自己实现的统一异常处理类，例如上面实现的`ExampleExceptionMapper`类
        deployment.getProviderClasses().add(RpcExceptionMapper.class.getName());
    ```
1. 在 META-INF 中新建 services 目录，在其中添加名为 `com.weibo.api.motan.protocol.restful.EndpointFactory`的文件 文件内容为自己实现的`EndpointFactory`实现类的全名
1. 通过 maven 发布这个 jar 包，在需要统一异常处理的工程中依赖，修改其 motan.xml 文件的 `<motan:protocol>` 标签，指定`endpointFactory`的值为`@SpiMeta`中指定的 name 属性的值

    ```
    <motan:protocol id="restfulProtocol" name="restful" endpointFactory="example"/>
    ```

### 通过 filter 处理 exception

github issue 中 motan 开发人员给出的建议： [在service端，怎么处理 统一异常处理机制](https://github.com/weibocom/motan/issues/433)

原理类似，这里简单介绍实现方式

#### 一个简单实现

1. 新建 Java 类实现`com.weibo.api.motan.filter.Filter`接口中的`Response filter(Caller<?> caller, Request request);`方法，通过 `@SpiMeta` 注解在类上设置 name 属性
1. 在`META-INF/services`目录中添加名为`com.weibo.api.motan.filter.Filter`的文件，内容为 Filter 接口实现类带有包路径的全名
1. 在`motan.xml`中修改对应的`<motan:service>`标签，添加属性`filter`，值为第一步中自定义实现类`@SpiMeta`中指定的 name 属性值
1. 业务逻辑抛出的原始异常可以通过下面方法获得

    ```
        Response response = caller.call(request);
        if (response.getException() != null) {
            Throwable cause = response.getException().getCause();
        }
    ```
