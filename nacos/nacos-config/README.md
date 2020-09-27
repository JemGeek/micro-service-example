# Nacos Config 配置中心

## 前言

在分布式系统中，由于服务数量巨多，为了方便服务 **配置文件统一管理，实时更新**，所以需要分布式配置中心组件。

`Spring Cloud Alibaba Nacos Config` 是 `Spring Cloud Config` 的替代方案。

`Nacos Config` 的存储配置功能为分布式系统中的外部化配置提供服务器端和客户端支持，可以在 `Nacos` 中集中管理 Spring Cloud 应用的外部属性配置。

## 项目介绍

### 项目主要功能

1. Nacos Config 单文件基本配置
2. Nacos Config 多配置文件, shared-configs 与 extension-configs 使用与区别
3. 使用 namespace 实现环境隔离
4. 使用 group 区分项目
5. shared-configs 比 extension-configs 优先级低

### `nacos-config-base`

此项目仅演示了 `naocs config` 的基本使用，动态刷新，以及使用 `namespace` 做环境隔离。

**添加依赖**

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

**添加配置**

`bootstrap.yml`

```yaml
spring:
  profiles:
    active: dev
  application:
    name: spring-cloud-nacos-config
  main:
    allow-bean-definition-overriding: true
  cloud:
    nacos:
      config:
        server-addr: 114.67.106.207:8848
        prefix: spring-cloud-nacos-config-base
        file-extension: yaml
        group: spring-cloud-example
```

`bootstrap-dev.yml`

```yaml
spring:
  cloud:
    nacos:
      config:
        namespace: 904174a3-d51f-43ed-a456-c4fd7386ecb3
```

**获取参数**

```java
@SpringBootApplication
public class NacosConfigBaseApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(NacosConfigBaseApplication.class, args);
        // 当动态配置刷新时，会更新到 Enviroment 中，因此这里每隔一秒中从 Enviroment 中获取配置
        String userName = applicationContext.getEnvironment().getProperty("user.name");
        String userAge = applicationContext.getEnvironment().getProperty("user.age");
        String active = applicationContext.getEnvironment().getProperty("profiles.active");
        System.err.println("profiles active: " + active + "; user name :" + userName + "; user age: " + userAge);
    }
}
```

### `nacos-config-multi`

> 多文件配置实践

配置文件如下：

```yaml
spring:
  application:
    name: spring-cloud-nacos-config-multi
  main:
    allow-bean-definition-overriding: true
  cloud:
    nacos:
      # 权限认证，nacos.core.auth.enabled=true 时需要添加
      username: nacos
      password: James2020+
      # 多配置文件
      config:
        server-addr: 192.168.9.17:8848
        namespace: 904174a3-d51f-43ed-a456-c4fd7386ecb3
        shared-configs:
          - data-id: test_one.yaml
            group: SPRING_CLOUD_EXAMPLE_GROUP
            refresh: true

          - data-id: test_two.yaml
            group: SPRING_CLOUD_EXAMPLE_GROUP
            refresh: true

        extension-configs:
          - data-id: spring-cloud-nacos-config-base.yaml
            group: SPRING_CLOUD_EXAMPLE_GROUP
            refresh: true
```




### `nacos-config-advanced`

此项目仅演示了 `naocs config` 的进阶使用，主要研究配置中心的环境隔离，多环境切换，是否支持数据源，Redis等配置。

**单文件研究结论**

可以使用 nacos config 配置数据源，Redis等配置，但这些配置，仅在启动时有效，启动后修改参数无效，需要重启才可。

优点：项目部署后，如需修改配置文件，仅在 nacos 中修改即可，但需要重启才能生效，可避免重新打包。
缺点：文件暴露在 Nacos Config 中，可能会导致不安全。

nacos config 更适合设置一些外部配置，可动态更新。

**多文件研究结论**


1. 如果本地 `bootstrap` 文件为 `yaml` 文件格式时，使用 `shared-configs` 或者 `extension-configs` 均不生效，需使用 `properties` 文件;
2. 如果 Nacos Config 中的配置文件为 *.yaml 文件格式时，

## 注意事项

1、 nacos config 相关的配置要配置在 `bootstrap.yml` 中。


