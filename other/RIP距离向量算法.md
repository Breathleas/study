## 基础
好消息传递快、坏消息传递慢
1. 默认，RIP使用简单的度量：通往目的站点所需经过的链路数。取值为1~15,数值16表示无穷大。
2. 使用UDP的520端口发送和接收RIP分组。
3. RIP 每隔30秒以广播形式发送一次路由信息，在邻居之间互传。
4. 防止广播风暴：后续分组做随机延时后发送。
5. 如果一个路由在180s内未被更新，相应的距离设置为无穷大：16，并从路由表中删除该表项。
6. RIP分组分为：
   请求分组
   响应分组
7. 路由收敛、直连跳数为1

## 参考
https://blog.csdn.net/hml666888/article/details/80153305
