# CPU之间的公式
总核数 = 物理CPU个数 X 每颗物理CPU的核数 
总逻辑CPU数 = 物理CPU个数 X 每颗物理CPU的核数 X 超线程数

# 查看物理CPU个数
```shell
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
```
# 查看每个物理CPU中core的个数(即核数)
```shell
cat /proc/cpuinfo| grep "cpu cores"| uniq
```
# 查看逻辑CPU的个数
```shell
cat /proc/cpuinfo| grep "processor"| wc -l
```
# 查看CPU信息（型号）
```shell
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
```
# 查看内 存信息
```shell
cat /proc/meminfo
```