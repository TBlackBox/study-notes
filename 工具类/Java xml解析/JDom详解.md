# 简介
使用方式：需要下载jdom对应的jar引入

<dependency>
    <groupId>org.jdom</groupId>
    <artifactId>jdom2</artifactId>
    <version>2.0.6</version>
</dependency>
 

# 使用场景

1、当需要知道整个文档结构

2、使用比较简单，比JDK自带的Document解析性能稍好

3、解析的树形结构操作比较灵活，比较简便

 

# JDOM操作xml常用类

Document：表示整个xml文档，是一个树形结构

Eelment：表示一个xml的元素，提供方法操作其子元素，如文本，属性和名称空间等

Attribute：表示元素包含的属性

Text：表示xml文本信息

XMLOutputter：xml输出流，底层是通过JDK中流实现

Format：提供xml文件输出的编码、样式和排版等设置

 

# JDOM生成XML
实现步骤：

第一步：先通过Eelment创建一个根结点
```java
Element rootElement = new Element("root");
```
 
第二步：把根结点添加到Document中
```java
Document doc = new Document(rootElement);
```

第三步：在根结点下添加一些子结点，构造成一个树形结构

第四步：创建XMLOutputter实例，输出xml文件，包括设置xml文件格式等
```java
XMLOutputter xmlOutput = new XMLOutputter();

xmlOutput.setFormat(Format.getRawFormat());

xmlOutput.output(doc, new FileOutputStream(file));
```

## 案列

## 引入Jar包
```
<dependency>
    <groupId>org.jdom</groupId>
    <artifactId>jdom</artifactId>
    <version>2.0.2</version>
</dependency>
```

## xml结构
```xml
<?xml version="1.0" encoding="utf-8" ?>
<users>
  <user id="0">
    <name>张三</name>
    <age>18</age>
    <sex>男</sex>
  </user>
  <user id="1">
    <name>吕雪</name>
    <age>24</age>
    <sex>女</sex>
  </user>
</users>
```

## java 列子
```java
import java.io.IOException;
import java.util.List;

import org.jdom2.Document;
import org.jdom2.Element;
import org.jdom2.JDOMException;
import org.jdom2.input.SAXBuilder;

public class JDomTest {
	public void parserXml(String fileName) {
	    SAXBuilder builder = new SAXBuilder();
	    try {
	      Document document = builder.build(fileName);
	      Element users = document.getRootElement();
	      List userList = users.getChildren("user");
	      for (int i = 0; i < userList.size(); i++) {
	        Element user = (Element) userList.get(i);
	        List userInfo = user.getChildren();
	        for (int j = 0; j < userInfo.size(); j++) {
	          System.out.println(((Element) userInfo.get(j)).getName() + ":" + ((Element) userInfo.get(j)).getValue());
	        }
	        System.out.println();
	      }
	    } catch (JDOMException e) {
	      e.printStackTrace();
	    } catch (IOException e) {
	      e.printStackTrace();
	    }
	 
	  }
	  public static void main(String[] args) {
	    JDomTest jdomTest = new JDomTest();
	    String path = Dom4jTest.class.getClassLoader().getResource("user.xml").getPath();
	    jdomTest.parserXml(path);
	     
	    }
}
```