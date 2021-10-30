# node流水线

## 注意

1. node 默认的是cnpm 如果有问题的话，可以换成npm

2. node 构建的话 ，如果有问题，可以清一下缓存,脚本这些并不复杂，尝试写就可以了

```
# input your command here
pwd
ls
echo -e "++++++++++开始install+++++++++++++++++"
npm cache clean -f
npm i
echo -e "++++++++++install完毕+++++++++++++++++"
npm run build:prod
```

