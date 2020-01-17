## note

hbase 重启后要等待会

hblog 保证正常情况0丢失数据  同时间写只能写一个 放在hdfs上

block cache 读缓存

memstore 写缓存,读也会有效果 

hbase快是因为block cache 和 memstore



hbase写的慢:

先调优linux,在hbase参数

