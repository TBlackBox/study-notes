
fetch 文档地址
https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API

个人博客
https://blog.csdn.net/hefeng6500/article/details/81456975

# 简介
Fetch API 提供了一个获取资源的接口（包括跨域请求）,fetch是一种HTTP数据请求的方式，是XMLHttpRequest的一种替代方案。fetch不是ajax的进一步封装，而是原生js。Fetch函数就是原生js，没有使用XMLHttpRequest对象。简单地说初ajax(XMLHttpRequest)  能获取资源，Fetch也能获取资源。

# 请求案例
下面的例子用的是response的转态作为成功的返回，将返回的数据转成json的，也能够指定为text（`response.text()`）,详细信息看下面介绍的对象。

## 不带参数的GET请求
```
//get 请求
function testGetRequest(){
	fetch("/menu/tree", {
		method: 'GET',
	}).then(response => {
		if (response.ok){
			response.json().then(message => {
				console.log(message);
				if (message.success){
					console.log(message.data);
				}else{
					alert(message.message);
				}
			});
		}
	});
}

```

## 带参数的GET请求
```
//带参数的get请求
function testGetRequestParame(id){
	fetch("/role/detail?id="+id, {
		method: 'GET',
	}).then(response => {
		if (response.ok){
			response.json().then(message => {
				console.log(message);
			});
		}
	});
}

```

## 不带参数的POST请求
```
//不带参数的POST请求
function testPostRequestNoParame(){
	fetch("/menu/tree", {
		method: 'POST',
	}).then(response => {
		if (response.ok){
			response.json().then(message => {
				console.log(message);
			});
		}
	});
}

```

## 带参数的POST 请求
```
//带参数的POST请求
function testPostRequestParame(id){
	const body = new URLSearchParams();
	body.set("id",id);
	fetch("/role/delete", {
		method: 'POST',
		body:body,
	}).then(response => {
		if (response.ok){
			response.json().then(message => {
				console.log(message);
			});
		}
	});
}
```


## 设置请求头的信息

在POST提交的过程中，默认的请求头是`Content-Type:text/plain;charset=UTF-8`,我们可以指定请求头。
```
//POST提交  指定请求头
function testPostRequestParame(id){
	const body = new URLSearchParams();
	body.set("id",id);
	fetch("/role/delete", {
		method: 'POST',
		headers: new Headers({
				// 指定提交方式为表单提交
		      'Content-Type': 'application/x-www-form-urlencoded' 
		}),
		body:body,
	}).then(response => {
		if (response.ok){
			response.json().then(message => {
				console.log(message);
			});
		}
	});
}
```

## 强制带cookie
默认情况下，fetch不会带cookie,需要带的话，需要加点东西，看例子就好了。
```
//强制带cookie 
function testGetCookieRequestNoParame(){
	fetch("/menu/tree", {
		method: 'GET',
		credentials: 'include' // 强制加入凭据头
	}).then(response => {
		if (response.ok){
			response.json().then(message => {
				console.log(message);
			});
		}
	});
}
```


# fetch的简单封装(需要更好的改进)
```

//test();
function test(){
	let data = GET('/menu/tree',null);
	data.then(response => {
		if(response.ok){
			console.log("response:",response.json());
		} 
	}).catch();
}

/**
 * GET请求
 * @param url 请求地址
 * @param data 请求参数
 */
async function GET(url, data) {
	if(data !=null){
		url += '?' + obj2String(data);
	}
	initObj = {
	  method: 'GET',
	  credentials: 'include'
	}
   return await fetch(url, initObj);
}
 

/**
 * POST请求
 * @param url 请求地址
 * @param data 请求参数
 */
async function POST(url, data = null,headers = null) {
	if(headers == null){
		headers = new Headers({
	        'Accept': 'application/json',
	        'Content-Type': 'application/x-www-form-urlencoded'
	      })
	}
	
	if(data != null){
		data = obj2String(date);
	}
	
	initObj = {
      method: 'POST',
      credentials: 'include',
      headers: headers,
      body: data
    }
  return await fetch(url, initObj);
}


/**
 * 将对象转成 a=1&b=2的形式
 * @param obj 对象
 */
function obj2String(obj, arr = [], idx = 0) {
  for (let item in obj) {
    arr[idx++] = [item, obj[item]]
  }
  return new URLSearchParams(arr).toString()
}

```