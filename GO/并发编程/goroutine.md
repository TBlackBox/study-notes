## 简介

goroutine（协程）的概览类似与线程，是有go运行时调度和管理。go程序会合理的把goroutine 分配给每个cpu,go 内置了调度和上下文切换机制。

 运行一个roroutine 非常简单，在调用函数前面加go 即可

#### goroutine调度

GPM是Go语言运行时（runtime）层面的实现，是go语言自己实现的一套调度系统。区别于操作系统调度OS线程。





# goroutine池

我们可以像java线程池一样，定义一个goroutine池，控制goroution的数量。

```go
//本质上是生产者消费者模型
// 可以有效控制goroutine数量，防止暴涨
// 需求：
// 1. 计算一个数字的各个位数之和，例如数字123，结果为1+2+3=6
// 2. 随机生成数字进行计算


//总结  
//1. main函数执行完毕 ，不会等里面的协程执行完毕都返回了
//2. 通过sync.WaitGroup 和Channel 需关闭通道  不然容易造成死锁
//3. goroutines池 可以对协程进行数量上的限制
//4. Channel通道的两种取值方法
package example

import (
	"fmt"
	"math/rand"
	"sync"
	"sync/atomic"
)

//将所有的协程都结束了  在结束main
var wg sync.WaitGroup

//用于打印计数
var pNum int32

//产生随机数的结构体
type job struct {
	id      int
	randNum int //产生的随机数
}

type result struct {
	job *job
	sum int //求和结果
}

//StartGoroutinedDemo  求属的和的入口
func StartGoroutinedDemo(jobNum int32, routinedNum int) {

	fmt.Printf("进入产生随机数之和入口,随机数数量：%d,工作协程数量：%d\n", jobNum, routinedNum)

	//定义两个管道
	jobChan := make(chan *job, 128)

	resultChan := make(chan *result, 128)

	//创建工作池  用于计算随机数各个位数的和
	createPool(routinedNum, jobChan, resultChan)

	wg.Add(2)
	//启动一个打印协程
	go print(jobNum, resultChan)

	//生成随机数
	genRandNum(jobNum, jobChan)

	// time.Sleep(time.Second * 10)
	fmt.Printf("进入产生随机数之和计算完成。。。。\n")

	wg.Wait()
}

func genRandNum(num int32, jobChan chan *job) {

	defer wg.Done()
	var i int32
	var h int = 0
	for i = 0; i < num; i++ {
		randNum := rand.Int()

		j := &job{
			id:      h,
			randNum: randNum,
		}
		h++
		jobChan <- j
	}
	close(jobChan)
}

//定义打印函数
func print(jobNum int32, resultChan chan *result) {
	defer wg.Done()
	//从结构结果管道里面获取值打印
	// for r := range resultChan {
	for {
		r, ok := <-resultChan
		if !ok {
			break
		}

		atomic.AddInt32(&pNum, 1)
		fmt.Printf("id:%d,  randNum:%d,  sum:%d\n", r.job.id, r.job.randNum, r.sum)
		//这点需要关闭 resultChan 通道  不关闭 会造成死锁  后面在找一哈有没得更好的优化方案
		if pNum == jobNum {
			close(resultChan)
		}
	}

	fmt.Printf("总共打印了：%d 条数据", pNum)

}

//工作池  主要负责计算
//num 为开几个协程数
func createPool(num int, jobChan chan *job, resultChan chan *result) {

	wg.Add(num)
	for i := 0; i < num; i++ {
		go work(jobChan, resultChan)
	}
}

func work(jobChan chan *job, resultChan chan *result) {

	defer wg.Done()

	for jobBody := range jobChan {
		randNum := jobBody.randNum

		var sum int
		for randNum != 0 {
			sum += randNum % 10
			randNum /= 10
		}

		//将结果封装到result里面
		r := &result{
			job: jobBody,
			sum: sum,
		}

		//将结果放入结果通道
		resultChan <- r
	}
}
```

