# 简介
API来源：在JDK中org.xml.*包下

# 使用场景

1、从上往下线性读取XML时

2、解析比较大的XML文档时

 

优点：读取速度快，内存消耗小

缺点：只能知道当前节点的信息，如需知道其它信息，需要自己编码实现

# SAX解析XML
实现步骤：

第一步：继承org.xml.sax.helpers.DefaultHandler类，重写几个方法

第二步：创建一个SAX解析工厂
```java
SAXParserFactory factory = SAXParserFactory.newInstance();
```
 
第三步：创建一个SAX转换工具
```java
SAXParser saxParser = factory.newSAXParser();
```

第四步：解析XML并打印到控制台
```java
saxParser.parse(inputFile, new ParseSAX());
```
# SAX生成XML
第一步：创建一个SAX转换工厂

SAXTransformerFactory factory = (SAXTransformerFactory)SAXTransformerFactory.newInstance();

 
第二步：创建一个TransformerHandler实例
```java
TransformerHandler handler = factory.newTransformerHandler();
```

第三步：创建一个handler转换器
```java
Transformer transformer = handler.getTransformer();

transformer.setOutputProperty(OutputKeys.INDENT, "yes");//换行
transformer.setOutputProperty(OutputKeys.ENCODING, "utf-8");//设置字符集

 ```

第四步：创建一个Result实例连接到XML文件
```
Result result = new StreamResult(new FileOutputStream(new File("E:\\sax.xml")));

handler.setResult(result);

 ```

第五步：创建一个属性实例，每次创建时，最好清除一下，如attr.clean();
```
AttributesImpl attr = new AttributesImpl();

attr.clear();

attr.addAttribute("", "", "attr", "", "root");

 ```

第六步：打开doc对象
``
handler.startDocument();
 ```

第七步：创建元素节点，并构建XML内容
```
handler.startElement(uri, 命名空间, 元素名, 属性); //没有则填null

 ```

第八步：设置标签内容
```java
characters(char ch[], int start, int length)

 ```

第九步：关闭doc对象，在指定的路径输出XML文件

# 案列
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

## java列子
```java
package xml;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;

import javax.xml.parsers.ParserConfigurationException;
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.DefaultHandler;


public class SaxTest {
	public void parserXml(String fileName) {
		SAXParserFactory saxfac = SAXParserFactory.newInstance();
		try {
			SAXParser saxparser = saxfac.newSAXParser();
			InputStream is = new FileInputStream(fileName);
			
			DefaultHandler defaultHandler = new MySAXHandler();
			
			saxparser.parse(is, defaultHandler);
		} catch (ParserConfigurationException e) {
			e.printStackTrace();
		} catch (SAXException e) {
			e.printStackTrace();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	public static void main(String[] args) {
		SaxTest saxTest = new SaxTest();
		String path = Dom4jTest.class.getClassLoader().getResource("user.xml").getPath();
		saxTest.parserXml(path);
	}
}

class MySAXHandler extends DefaultHandler {
	boolean hasAttribute = false;
	Attributes attributes = null;

	public void startDocument() {
		 System.out.println("文档开始打印了");
	}

	public void endDocument() {
		 System.out.println("文档打印结束了");
	}

	public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
		if (qName.equals("users")) {
			return;
		}
		if (qName.equals("user")) {
			return;
		}
		if (attributes.getLength() > 0) {
			this.attributes = attributes;
			this.hasAttribute = true;
		}
	}

	public void endElement(String uri, String localName, String qName) {
		if (hasAttribute && (attributes != null)) {
			for (int i = 0; i < attributes.getLength(); i++) {
				System.out.print(attributes.getQName(0) + ":" + attributes.getValue(0));
			}
		}
	}

	public void characters(char[] ch, int start, int length) {
		System.out.print(new String(ch, start, length));
	}
}

```