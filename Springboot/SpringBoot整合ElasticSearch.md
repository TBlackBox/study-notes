
# 简介
SpringBoot默认支持Jest和SpringDate ElasticSearch两种方式，SpringBoot默认使用SpringData来操作ElasticSearch，
 
[elasticsearch文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)  

[githb地址](https://github.com/elastic/elasticsearch)

# Jest方式操作ES
在SpringBoot的JestAutoConfiguration自动配置类里面，默认是不生效的，需要引入JestClient类。Jset相当于通过http的形式炒作ES.

1. 导入依赖,注意版本要和es的版本相对于。
```java
<!-- https://mvnrepository.com/artifact/io.searchbox/jest -->
<dependency>
    <groupId>io.searchbox</groupId>
    <artifactId>jest</artifactId>
    <version>5.3.3</version>
</dependency>
```

2. 配置地址
```java
spring:
  elasticsearch:
    jest:
      uris: https://localhost:9200
```
3. 通过JestClient操作ES
在JestAutoConfiguration里注入了JestClient，通过他可以操作es.
```java
	@Autowired
	JestClient jestClient;

	@Autowired
	public void testJestClient() throws IOException {
		//1. 构建一个索引功能
		// user是要存的参数  test 是索引  tag1 是类型
		Index index = new Index.Builder("user").index("test").type("tag1").build();
		//2. 执行
		jestClient.execute(index);

		//搜索
		//查询表达式
		String json = "{\n" +
				"    \"query\" : {\n" +
				"        \"match_all\" : {}\n" +
				"    }\n" +
				"}'";
		//构建搜索功能
		Search search = new Search.Builder(json).addIndex("test").addType("type").build();
		SearchResult  searchResult = jestClient.execute(search);
		//通过searchResult获取信息
		System.out.println(searchResult.getJsonObject());
	}
```


# SpringDate的方式操作ES
SpringBoot的ES自动配置类ElasticsearchDataAutoConfiguration注入了ElasticsearchTemplate进行ES的交互。我们只需要引入相关配制即可，第二总方式定义一个类，继承ElasticsearchRepositoriesAutoConfiguration，就像操作jpa一样，可通过名字来操作ES,详情查看[soringdate文档](https://spring.io/projects/spring-data-elasticsearch).

1. pom.xml里面引入启动器
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

2. 配置节点名字和节点地址
```java
spring:
  data:
    elasticsearch:
      #节点名字
      cluster-name: elasticsearch
      cluster-nodes: 192.168.0.110:9300
```

**特别注意：**
启动好可能会出错，可能是又要springboot里面，springDate基础elsaticsearcs的版本和安装的es 不匹配，要么就升级springDate,要么就从新安装es对应的版本。可以参考springData的对应文档
[对应文档](https://docs.spring.io/spring-data/elasticsearch/docs/3.2.0.RC3/reference/html/#preface.versions)

3. 下面就可以通过ElasticsearchTemplate 来操作ES了  

[操作建官方文档](https://docs.spring.io/spring-data/elasticsearch/docs/3.1.10.RELEASE/reference/html/)