#发布日志
###2015年8月3日Inception2.0.13-beta发布
1. 新增使用event对象的检查
2. 修改多表更新时，审核通过，执行失败的问题
3. 在表列定义为int(2)类似的情况下，会导致在设置Binlog解析记录长度时错误导致内存correption的问题
4. 在解析binlog出错之后，没有再去dump导致直接读取网络引起一直阻塞的问题
5. 修改对线上数据库操作时，数据库名字没有加"\`"导致报错的问题
6. 修改设置BLOB/TEXT为NOT NULL时报错级别为警告
7. 禁止在语句中直接使用@uservar类型的表达式
8. 修复Inception备份时Binlog解析错乱的问题
9. 增加对创建索引及新建表中，索引长度超过767的检查
10. 修改在生成ALTER回滚语句出错的问题

###2015年6月1日Inception2.0.3-beta发布
新增：  
1. 实现OSC执行的取消功能  
2. 在审核前获取一些线上参数来辅助审核，比如参数：explicit_defaults_for_timestamp
修复：  
1. 在Inception没有dump binlog权限的情况下，备份时Inception会陷入死循环的问题  

###2015年5月22日Inception2.0.0-beta发布