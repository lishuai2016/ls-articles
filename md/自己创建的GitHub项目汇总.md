# 自己创建的GitHub项目汇总


# didi
一个阿里校招的面试题

题目：模拟用户滴滴打车场景，根据用户的经纬度信息查找距离用户最近的车辆。
接受一个带有经纬度信息的http请求，查找距离该经度最近的车辆。


方案：
根据我查得的信息，滴滴司机端那边会定时刷新自己的地理，然后上传到服务器；
由于没有数据，我这里通过以用户的位置坐标为基准，在其附近随机生成30个点代表附近的车辆，
每次刷新由于是随机生成的点，动态模拟用户身边的车辆；
我对这30个点，根据其与用户的距离进行排序，并在地图上标注出来1~30，点击红色的标注点还可以看到距离用户的具体距离数据。


注释：要是考虑真实司机经纬度数据从数据库取的话，我的思路是根据客户传回来的经纬度，
以该客户的经纬度点为圆心，以R为半径，取出数据库中的附近所有车辆经纬度信息，然后根据距离的远近进行地图上的标注。


地址：
https://github.com/lishuai2016/didi.git

# ls
自己的课设demo

https://github.com/lishuai2016/ls


# ls-bigdata
大数据各个组件的开发学习
其中包括：
zookeeper
Hadoop
MapReduce
flume
kafka
HBASE
storm
spark


https://github.com/lishuai2016/ls-bigdata

升级版
https://github.com/lishuai2016/ls-bigdata-learn

# ls-elasticsearch
elasticsearch实践demo
这只是一个dubbo的服务提供端
https://github.com/lishuai2016/ls-elasticsearch

服务的消费端
ls-dubbo-consumer
https://github.com/lishuai2016/ls-dubbo-consumer

# ls-es-hbase
一个类似百度的搜索引擎，通过es和hbase实现
https://github.com/lishuai2016/ls-es-hbase


# ls-py
自己的Python学习分支
https://github.com/lishuai2016/ls-py

# ls-spark
https://github.com/lishuai2016/ls-spark
spark学习

# ls-springboot-learn
项目地址：
https://github.com/lishuai2016/ls-springboot-learn

# ls-springcloud-learn
项目地址：
https://github.com/lishuai2016/ls-springcloud-learn

# ls-springconfig-repo
地址：https://gitee.com/2016shuai/ls-springconfig-repo
springcloud的远程仓库


# WebRTC01
一个简单的WebRTC实例
https://github.com/lishuai2016/WebRTC01


# 源码学习项目

## 1、netty
GitHub地址：https://github.com/lishuai2016/ls-netty
本地位置：Desktop\netty\ls-netty\netty-4.1



## 2、dubbo
GitHub地址：https://github.com/lishuai2016/incubator-dubbo
本地地址：D:\opencode\incubator-dubbo