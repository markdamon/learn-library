# OAuth2(GO)-Apollo-Zuul/Eureka/Ribbon/Hystrix-CAT/Prometheus/grafana

## 一、概念模型

项目地址：[PiggyMetrics](https://github.com/spring2go/piggymetrics)

## 二、环境搭建及注意事项

### 1. 统一授权认证中心Gravitee OAuth2

- [go环境搭建](https://github.com/spring2go/gravitee_lab/tree/master/lab01)

  遇到依赖编译错误

  a）、如vs code编辑器中提示golang.org/x/tools/cmd/guru依赖错误，可以通过go get -u -v golang.org/x/tools/cmd/guru来下载依赖模块。

  b）、如果上面的地址下载出现失败，可以通过离线的方式来下载，下载地址以golang.org/x/tools/cmd/guru为例：https://github.com/golang/tools/tree/master/cmd/guru

  下载以后存放在${GOPATH}/src/golang.org/x/tools/cmd/guru/目录下。

- [初始化Gravitee项目](https://github.com/spring2go/gravitee_lab/tree/master/lab02)

- 修改端口为5000

### 2. 集中配置Apollo

- [搭建apollo环境](https://github.com/spring2go/apollo_lab/tree/master/lab01)

  启动完成，但在apollo admin管理页面添加项目以后出现配置错误。

  检查端口8090是否被占用（可以检查一下apollo admin的启动状态：http://localhost:8070/system_info.html ），mac/linux查看端口占用命令：lost -i tcp:8090，找到进程id并kill -9 id杀掉。

- 将项目配置文件录入到apollo中。配置地址：[config](https://github.com/spring2go/piggymetrics/tree/master/config)

### 3. 监控反馈CAT

- [cat环境搭建](https://github.com/spring2go/cat_lab/tree/master/lab01)

  服务启动过程注意打开/data/appdatas/cat/cat_20190820.log日志查看日志信息，保证服务启动没有错误，且基础服务启动的过程中也会在这个文件中输出相关接入成功的日志。

- [修改端口为8081](https://github.com/spring2go/case_study_lab/tree/master/lab04)

### 4. 基础服务Zuul/Eureka/Ribbon/Hystrix

- [搭建数据库mongodb](https://github.com/spring2go/case_study_lab/tree/master/lab01)

  mac对应的配置地址：/usr/local/etc/mongod.conf

```properties
security:
  authorization: enabled
```

- 按顺序启动项目
  1. registry
  2. account-service
  3. statistics-service
  4. notification-service
  5. gateway

### 5. 监控数据存储Prometheus

- [搭建prometheus环境](https://github.com/spring2go/prom_lab/tree/master/lab01)

- 添加配置

  修改prometheus配置文件：prometheus.yml
```properties
  - job_name: 'piggy-metrics'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:4000']
```

- 不需要重启prometheus服务即可动态加载配置

  启动prometheus时需要添加参数，完整启动方式：./prometheus --web.enable-lifecycle

  修改完prometheus.yml以后get访问：http://localhost:9090/-/reload

### 6. 监控数据展示Grafana

- [grafana环境搭建](https://github.com/spring2go/prom_lab/tree/master/lab02)

- 挑选已有spring boot的dashboard

  官方提供免费dashboard地址：https://grafana.com/grafana/dashboards ，DataSource选择prometheus，category选择web servers，search within this list填入：spring boot。在右侧选择下次次数比较多的：*Spring Boot Statistics*，然后Copy ID to Clipboard：6756

- 导入dashboard=6756

  通过Grafana的**+**图标导入(**Import**) Spring Boot Statistics dashboard：

  - grafana id = **6756**
  - 注意选中`prometheus`数据源
  
- 该dashboard=6756的application需要在项目写入一下代码：

```java
@Bean
MeterRegistryCustomizer<MeterRegistry> metricsCommonTags(@Value("${app.id}") String appId) {
  return registry -> registry.config().commonTags("application", appId);
}
```
- 该dashboard=6756在展示的时候需要调整grafana的变量

  进入dashboard=6756面板-设置，在Variables中，

  修改application（spring.application.name/app.id）为：label_values(application)，

  修改instance（一般为IP:PORT）为：label_values(jvm_memory_used_bytes{application="$application"}, instance)

## 三、项目总结

- Gravitee OAuth2可以尝试添加Apollo进行配置管理，参考Apollo[其它语言客户端接入指南](https://github.com/ctripcorp/apollo/wiki/%E5%85%B6%E5%AE%83%E8%AF%AD%E8%A8%80%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97)

- Gravitee OAuth2可以尝试Prometheus添加监控，参考[CLIENT LIBRARIES](https://prometheus.io/docs/instrumenting/clientlibs/)