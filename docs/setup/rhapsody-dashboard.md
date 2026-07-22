#### Rhapsody Dashboard部署

##### 安装后效果图

![image-20220919164827055](/docs-note-rhapsody/assets/images/image-20220919164827055.png)

##### 安装准备

| 序号 | 程序文件                                 | 描述       |
| ---- | ---------------------------------------- | ---------- |
| 1    | liaozhiming/rhapsody_dashboard:2.1.3.260421 | 监控程序容器镜像   |

##### 安装步骤

1. 安装监控程序
2. 配置监控程序

##### 程序配置

   
1. 配置监控引擎

   ```python
   # 修改配置文件 dashboard\config\engine.properties
   # 分组名称,如果需要多个分组,以此类推,Group.x.Name
   Group.1.Name = Test
   Group.1.Default = true
   
   # 监控的引擎地址,以此类推,配置多个监控地址 Engine.x.Hostname
   Engine.2.Hostname = 172.16.33.147
   Engine.2.Port = 8444
   Engine.2.Username = rest_api_rhapsody_dashboard
   Engine.2.Password = REN600c3586c33793b3dbeb40aa0e4b4c02135e99e5321019a52d3bc7f228d69c86e2fc5c16dabe961b62b6c
   Engine.2.Thumbprint =51 AF 12 6B 02 A3 FA 63 39 58 C6 C3 00 FC 00 B3 8D 64 79 A3 # 证书指纹
   Engine.2.Disabled = false
   Engine.2.Polling = 120
   Engine.2.Groups = 2 # 分组的名称
   Engine.2.CPUWarning = 75
   Engine.2.CPUAlarm = 95
   ```
2. 启动命令
  ```sh
  # 注意：需要在配置文件dashboard.properties中取消配置项Dashboard.ssl.ciphers；同时重启容器服务
  
  docker run -d --name rhapsody-dashboard-beta -e TZ=Asia/Shanghai -p 8250:8250 --restart unless-stopped liaozhiming/rhapsody_dashboard:2.1.3.260421
  ```

!!! success "证书指纹(Thumbprint)可通过以下路径文件中进行获取:"
      **服务端-windows端**: 示例`C:\Program Files\Rhapsody\Rhapsody Engine 7\rhapsody\ide.thumbprint.txt` <br>
      **服务端-centos端**: 示例`/home/rhapsody/rhapsody/rhapsody-engine-7/rhapsody/ide.thumbprint.txt` <br>
