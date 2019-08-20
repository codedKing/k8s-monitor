# k8s-monitor

#### 介绍
采用Prometheus+ Grafana架构方案来监控k8s，Prometheus存储监控数据，而Grafana展示监控数据。Prometheus是由SoundCloud开发的开源监控报警系统和时序列数据库(TSDB)。Prometheus使用Go语言开发，是Google BorgMon监控系统的开源版本。Prometheus的优秀特性如下：多维度数据模型。灵活的查询语言。不依赖分布式存储，单个服务器节点是自主的。通过基于HTTP的pull方式采集时序数据。可以通过中间网关进行时序列数据推送。通过服务发现或者静态配置来发现目标服务对象。支持多种多样的图表和界面展示，比如Grafana等

#### 软件架构
![软件架构说明](https://images.gitee.com/uploads/images/2019/0820/092914_b5549966_435593.png "图片2.png")

架构说明：

1. Prometheus Server：核心组件，负责收刮和存储时序数据（time series data），并且提供查询接口。

2. Jobs/Exporters：客户端，监控并采集指标，对外暴露HTTP服务（/metrics）。

3. Pushgateway：针对push系统设计，Short-lived jobs定时将指标push到Pushgateway，再由Prometheus Server从Pushgateway上pull。

4. Alertmanager：报警组件，根据实现配置的规则（rule）进行响应，例如发送邮件；目前未配置

5. Web UI：Prometheus内置一个简单的Web控制台，可以查询指标，查看配置信息或者Service Discovery等，实际工作中，查看指标或者创建仪表盘通常使用Grafana，Prometheus作为Grafana的数据源进行展示

#### 监控收集需要考虑的内容

数据持久化：Prometheus为时序性数据库，数据存放本地，需要创建pv和pvc进行持久化

配置文件持久化：配置文件存放到本地，每次更新就可。
#### 安装教程

1. 创建命名空间namespace.yaml和node-exporter.yaml 并执行
    kubectl  create -f namespace.yaml
	kubectl  create -f node-exporter.yaml
2. 创建Prometheus和Grafana的pv，pvc 并执行
	kubectl  create -f prometheus-pv.yaml
	kubectl  create -f prometheus-pvc.yaml
	kubectl  create -f grafana-pv.yaml
	kubectl  create -f grafana-pvc.yaml
3. 配置prometheus的configmap.yaml，添加监控的机器如：
   - job_name: '172.31.10.192'
      static_configs:
        - targets: ['172.31.10.192:9100']
          labels:
            name: 172.31.10.192
            instance: 172.31.10.192
            env: prod
	执行配置 kubectl create -f configmap.yaml
	3.1创建prometheus启动需要的SA账户，只有具体有特殊的权限，prometheus应用才可以读取API Sever和POD的相关数据
	kubectl create -f rbac-setup.yaml
	3.2创建Prometheus的svc，用于访问prometheus。使用NodePort类型
	kubectl create -f  prometheus.svc.yml
	3.3 配置prometheus的部署文件prometheus.deploy.yml 并执行
	kubectl create -f  prometheus.svc.yml
4.创建grafana的svc 并执行
	kubectl  create -f grafana-svc.yaml
5.创建grafana的ingress 并执行
	kubectl  create -f grafana-ing.yaml
6.创建grafana的deploy 并执行
	kubectl  create -f grafana-deploy.yaml
7.执行需要监控机器的内部指标（cpu,内存等）转到node_exporter-0.18.1.linux-amd64目录
    ./node_exporter &


#### 参与贡献

1. Fork 本仓库
2. 新建 Feat_xxx 分支
3. 提交代码
4. 新建 Pull Request


#### 码云特技

1. 使用 Readme\_XXX.md 来支持不同的语言，例如 Readme\_en.md, Readme\_zh.md
2. 码云官方博客 [blog.gitee.com](https://blog.gitee.com)
3. 你可以 [https://gitee.com/explore](https://gitee.com/explore) 这个地址来了解码云上的优秀开源项目
4. [GVP](https://gitee.com/gvp) 全称是码云最有价值开源项目，是码云综合评定出的优秀开源项目
5. 码云官方提供的使用手册 [https://gitee.com/help](https://gitee.com/help)
6. 码云封面人物是一档用来展示码云会员风采的栏目 [https://gitee.com/gitee-stars/](https://gitee.com/gitee-stars/)