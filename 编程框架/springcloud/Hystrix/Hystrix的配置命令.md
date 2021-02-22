![](../../../\sources\springcloud\Hystrix的45.png)

![](../../../\sources\springcloud\Hystrix的46.png)

![](../../../\sources\springcloud\Hystrix的47.png)

![](C:/work/code/study-notes/编程框架/springcloud/图片/Hystrix的48.png)

![](../../../\sources\springcloud\Hystrix的49.png)



# 熔断的整体流程
```


1请求进来,首先查询缓存,如果缓存有,直接返回
  	如果缓存没有,--->2
2,查看断路器是否开启,如果开启的,Hystrix直接将请求转发到降级返回,然后返回
  	如果断路器是关闭的,
				判断线程池等资源是否已经满了,如果已经满了
  					也会走降级方法
  			如果资源没有满,判断我们使用的什么类型的Hystrix,决定调用构造方法还是run方法
        然后处理请求
        然后Hystrix将本次请求的结果信息汇报给断路器,因为断路器此时可能是开启的
          			(因为断路器开启也是可以接收请求的)
        		断路器收到信息,判断是否符合开启或关闭断路器的条件,
				如果本次请求处理失败,又会进入降级方法
        如果处理成功,判断处理是否超时,如果超时了,也进入降级方法
        最后,没有超时,则本次请求处理成功,将结果返回给controller
         
```