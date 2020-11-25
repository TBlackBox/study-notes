# 简介
非常灵活的java解析xml工具，推荐使用

[git地址](https://dom4j.github.io/)

[官网](https://dom4j.github.io/)


DOM中的核心概念就是节点，在XML文档中的元素、属性、文本等，在DOM中都是节点!


# 常用api
1. SaxReader 对象
 - read(...) 加载执行xml文档
2. Document 对象
 - getRootElement() 获取根元素
3. Element 对象
  - enements(...)获取指定名称的所有子元素，也可以不指定名称。
  - element(...) 获取指定名称的第一个子元素，可以不指定名称
  - getName() 获取当前元素的元素名
  - attributeValue(...)获取指定属性名的属性值
  - elementText(...)获取指定名称子元素的文本值
  - getText() 获取当前元素的文本内容 
# 使用列子

## 添加依赖包
```
<dependency>
            <groupId>dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>1.6.1</version>
</dependency>
```
注意：这里是低版本的依赖包，高本版的请看github地址里面的文档
## xml 文件
```
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

## xml是字符串的时候
如果您有一些XML作为字符串，则可以使用辅助方法DocumentHelper.parseText（）将其再次解析回Document中。
```
String text = "<person> <name>James</name> </person>";
Document document = DocumentHelper.parseText(text);
```
## Java例子
更多的解析方法，看github文档
```
import java.io.File;
import java.util.Iterator;

import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

public class Dom4jTest {

	public void parserXml(String fileName) {
	    File inputXml = new File(fileName);
	    SAXReader saxReader = new SAXReader();
	 
	    try {
	      Document document = saxReader.read(inputXml);
	      Element users = document.getRootElement();
	      for (Iterator i = users.elementIterator(); i.hasNext();) {
	        Element user = (Element) i.next();
	        System.out.println("id:" + user.attributeValue("id"));
	        for (Iterator j = user.elementIterator(); j.hasNext();) {
	          Element node = (Element) j.next();
	          System.out.println(node.getName() + ":" + node.getText());
	        }
	        System.out.println();
	      }
	    } catch (DocumentException e) {
	      System.out.println(e.getMessage());
	    }
	  }
	
	public static void main(String[] args) {
		Dom4jTest domj4= new Dom4jTest();
		
		String path = Dom4jTest.class.getClassLoader().getResource("user.xml").getPath();
	    System.out.println(path);
	    //获取文件路径
	    domj4.parserXml(path);
	}
}

```