#### **数据库操作**

##### 解析输入消息

| 字段                                                         | 含义/示例                                                    |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Rhapsody全局变量                                             | $(myVariable)                                                |
| 获得上一个数据集字段数据或者数据库关键列或者存储过程select的值 | @id                                                          |
| EDI消息中的字段使用@                                         | @PID.PatientName[0].FamilyName.Surname                       |
| XML消息中的字段使用`,并且使用完整路径,                       | 示例:个人信息查询服务提取本地患者ID<br>  &#96;/*/controlActProcess/queryByParameter/parameterList/id/@extension&#96; |
| 列自上一个结果使用@                                          | @PatientName                                                 |
| 存储过程输出参数使用#                                        | #newPatientId                                                |
| 消息主体                                                     | **$messageContent**                                          |
| 动态表名/列名;                                               | $$tbl_name&nbsp;&nbsp;;需要开启允许动态查询的配置(**Options**) |

!!! success "数据库dbe过滤器支持多条sql语句查询,可以减少一个路由使用多次数据过滤器查询"

!!! success "数据库db1过滤器支持循环遍历xml节点,选择遍历节点后,格式示例&nbsp;{==/root/data==}&nbsp;;子语句无需标注循环节点索引值"

!!! warning "关键列只针对数据库通信点生效,对于过滤器不起作用"

!!! warning "dbe数据库过滤器不应处理查询大批量数据,否则导致引擎GC Pause;<br>{==dbe数据库不适用于设置消息属性;设置消息属性请改用dbl数据库过滤器;==}"

!!! tip "允许dbe数据库过滤器查询结果为空不报错,请在dbe数据库过滤器配置xml界面添加下面内容:<br>{==&lt;allowEmptyResult&gt;ture&lt;/allowEmptyResult&gt;==}"

???+ danger highlight blink "除以下支持外,其它需要自行下载jdbc驱动"
    - [x] **SQL Server**<br>
    - [x] **Oracle**

!!! example  "自定义驱动程序存放位置"
     **服务端-windows端(需要重启引擎)**: 示例`C:\Program Files\Rhapsody\Rhapsody Engine 6\rhapsody\data\lib` <br>
     **服务端-centos端(需要重启引擎)**: 示例`/home/rhapsody/rhapsody/rhapsody-engine-6/rhapsody/data/lib` <br>
     **IDE端**: 示例 C:\Program Files (x86)\Rhapsody\Rhapsody IDE 6\Rhapsody IDE\DatabaseStructureReader

#### Access数据库

```python
# 驱动
Access_JDBC30.jar
# java驱动类名 driver class name
com.hxtt.sql.access.AccessDriver
# jdbc url;请注意,如果是ide则为ide能访问的地址,如果是服务端则填服务端的地址
# windows下连接方法
jdbc:access:/D:/KHdata/Database/Upload.mdb
# centos下连接方法示例
jdbc:access:////opt/dicom/Upload.mdb
# 通过测试Rhapsody版本: 6.7
```

#### MongoDB数据库

```python
# 驱动
mongodb_unityjdbc_full.jar
# java驱动类名 driver class name
mongodb.jdbc.MongoDriver
# 连接方法示例
jdbc:mongo://10.88.10.72:27077/esb_msg
# 通过测试Rhapsody版本: 6.7
```

#### Cache数据库

```python
# 驱动
cachejdbc.jar
# java驱动类名 driver class name
com.intersys.jdbc.CacheDriver
# 连接方法示例
jdbc:Cache://10.88.10.72:27077/test
# 通过测试Rhapsody版本: 6.7
```

#### MySQL数据库

```python
# 驱动
mysql-connector-java-8.0.21.jar
# java驱动类名 driver class name
com.mysql.cj.jdbc.Driver
# 连接方法示例
jdbc:mysql://DatabaseServer/PatientRecords
# 测试语句
select 1 code
# 常见问题-1 ：MySQL8报错：Public Key Retrieval is not allowed
# 添加参数： jdbc:mysql://localhost/yourDatabase?allowPublicKeyRetrieval=true&useSSL=false
# 通过测试Rhapsody版本: 6.7
```

### Redis

```python
# 驱动
redis-jdbc-driver-1.4.jar
# java驱动类名 driver class name
jdbc.RedisDriver
# 连接方法示例;用户名:default, 密码
jdbc:redis://172.17.0.2:6379
# 测试语句
get test
# 通过测试Rhapsody版本: 7.1
```

