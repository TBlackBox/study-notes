1. 下载地址

   ```
   https://github.com/alibaba/nacos/releases
   ```

   

2. 解压

```
unzip nacos-server-$version.zip 或者 tar -xvf nacos-server-$version.tar.gz
 cd nacos/bin
```



## 启动服务器

### Linux/Unix/Mac

启动命令(standalone代表着单机模式运行，非集群模式):

```
sh startup.sh -m standalone
```

如果您使用的是ubuntu系统，或者运行脚本报错提示[[符号找不到，可尝试如下运行：

```
bash startup.sh -m standalone
```

