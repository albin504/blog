# 背景

最近把分布式调度系统xxl-job部署到生产环境了，用于团队内crontab脚本的调度。

生产环境的服务，监控是很重要一环。今天就详细讲一下监控配置过程。



# 监控步骤

## spring boot 项目中，开启actuator组件，并整合Prometheus。这样，就输出了监控指标。

效果如下：

通过$host/actuator/prometheus地址，输出prometheus 格式的指标数据：

```
# HELP jvm_memory_max_bytes The maximum amount of memory in bytes that can be used for memory management
# TYPE jvm_memory_max_bytes gauge
jvm_memory_max_bytes{area="heap",id="PS Survivor Space",} 1.08003328E8
jvm_memory_max_bytes{area="heap",id="PS Old Gen",} 7.158628352E9
jvm_memory_max_bytes{area="heap",id="PS Eden Space",} 3.359113216E9
jvm_memory_max_bytes{area="nonheap",id="Metaspace",} -1.0
jvm_memory_max_bytes{area="nonheap",id="Code Cache",} 2.5165824E8
jvm_memory_max_bytes{area="nonheap",id="Compressed Class Space",} 1.073741824E9

# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="heap",id="PS Survivor Space",} 7.0486256E7
jvm_memory_used_bytes{area="heap",id="PS Old Gen",} 1.340948E7
jvm_memory_used_bytes{area="heap",id="PS Eden Space",} 9.105228E7
jvm_memory_used_bytes{area="nonheap",id="Metaspace",} 5.0553672E7
jvm_memory_used_bytes{area="nonheap",id="Code Cache",} 3.3104256E7
jvm_memory_used_bytes{area="nonheap",id="Compressed Class Space",} 6141752.0

jvm_threads_live_threads 233.0
jvm_threads_peak_threads 241.0

# HELP logback_events_total Number of error level events that made it to the logs
# TYPE logback_events_total counter
logback_events_total{level="warn",} 0.0
logback_events_total{level="debug",} 0.0
logback_events_total{level="error",} 0.0
logback_events_total{level="trace",} 0.0
logback_events_total{level="info",} 385.0

# TYPE http_server_requests_seconds summary
http_server_requests_seconds_count{exception="None",method="GET",outcome="SUCCESS",status="200",uri="/actuator/prometheus",} 3007.0
http_server_requests_seconds_sum{exception="None",method="GET",outcome="SUCCESS",status="200",uri="/actuator/prometheus",} 18.204923904
http_server_requests_seconds_count{exception="None",method="GET",outcome="SUCCESS",status="200",uri="/actuator/threaddump",} 3.0
http_server_requests_seconds_sum{exception="None",method="GET",outcome="SUCCESS",status="200",uri="/actuator/threaddump",} 0.468137103
http_server_requests_seconds_count{exception="None",method="GET",outcome="SUCCESS",status="200",uri="/actuator",} 15.0
http_server_requests_seconds_sum{exception="None",method="GET",outcome="SUCCESS",status="200",uri="/actuator",} 0.129559821
http_server_requests_seconds_count{exception="None",method="GET",outcome="CLIENT_ERROR",status="404",uri="/**",} 2.0
http_server_requests_seconds_sum{exception="None",method="GET",outcome="CLIENT_ERROR",status="404",uri="/**",} 0.011090137

```

以上是我输出的部分指标数据，包含了以下类型的监控指标：
1. jvm最大内存、当前占用的内存。 
2. 当前总的活跃线程数
3. 错误日志数量
4. 每秒http请求数

我们要保证服务的可靠运行，这些类型的指标监控，都是很关键的。


现在，通过actuator以及prometheus，我们轻易就获取到了这些指标数据。

## 利用prometheus进行监控报警

生产环境我部署的xxl-job，都是多节点冗余部署，避免单点故障。

利用prometheus指标表达式，就可以轻易完成以下类型的监控报警：
1. 1个节点挂了
2. jvm内存快超了
3. 产生error log了
4. 线程数暴增，可能出现故障了。
。。。。。。

这些异常情况，都可以进行报警。

## 利用grafana dashboard，查看系统趋势图，进行数据分析。

利用prometheus，能完成异常情况的报警。

另外，我想看到服务的健康状态趋势图，如：
- 看jvm内存占用变化趋势
- 看error log数量变化趋势图

这时候就用到grafana了。

### 我配置的grafana效果图



