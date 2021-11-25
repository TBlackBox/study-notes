# java的构建

## 需要注意的问题

1. 环境

   流水线代码是现在docker里面构建的，构建完后，会把制品上传到主机部署对应的下载路径中

2. 打包路径

   注意这个地方，也就是定制制品里面的内容，这点可以把需要的文件或者文件夹打包到制品里面。

3. 主机部署 

   这点实现通用的启动脚本都可以了

   ```shell
   
   echo -e "------------包路径--------------"
   pwd
   echo -e "包的路径输出完毕"
   tar zxvf /home/admin/app/package.tgz -C /home/admin/app/
   sh /home/admin/app/bin/deploy.sh
   echo -e "++++++++代码重启完毕++++++++++++++"
   
   # 部署脚本会在部署组的每台机器上执行。一个典型脚本逻辑如下：先将制品包（在下载路径中配置的下载路径）解压缩到指定目录中，再执行启动脚本（通常在代码中维护，如示例中deploy.sh）。关于这个例子的详细解释见 https://help.aliyun.com/document_detail/153848.html \n\n # tar zxvf /home/admin/app/package.tgz -C /home/admin/app/\n # sh /home/admin/app/deploy.sh restart\n # 如果你是php之类的无需制品包的制品方式，可以使用git clone 或者 git pull将源代码更新到服务器，再执行其他命令 \n # git clone ***@***.git\n
   ```

   4. 调试触发 要配置webhook

