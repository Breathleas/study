## 主要类作用
1. OkHttpClient对象：为网络请求执行的一个中心，它会管理连接池，缓存，SocketFactory，代理，各种超时时间，DNS，请求执行结果的分发。
2. Request对象：用于描述一个HTTP请求，比如：请求的方法是GET还是POST，请求的URL，请求的header，请求的body，请求的缓存策略等。

## 几个Interceptor的职责
RetryAndFollowUpInterceptor --->创建StreamAllocation对象，处理http的redirect，出错重试。对后续Interceptor的执行的影响：修改request及StreamAllocation。
BridgeInterceptor-------------->补全缺失的一些http header。对后续Interceptor的执行的影响：修改request。
CacheInterceptor-------------->处理http缓存。对后续Interceptor的执行的影响：若缓存中有所需请求的响应，则后续Interceptor不再执行。
ConnectInterceptor------------>借助于前面分配的StreamAllocation对象建立与服务器之间的连接，并选定交互所用的协议是HTTP 1.1还是HTTP 2。对后续Interceptor的执行的影响：创建了httpStream和connection。
CallServerInterceptor----------->处理IO，与服务器进行数据交换。对后续Interceptor的执行的影响：为Interceptor链中的最后一个Interceptor，没有后续Interceptor。
