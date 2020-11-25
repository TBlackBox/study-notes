# 简介
Dom解析xml

# 案列
xml结构
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

## java代码
```
package xml;

import java.io.FileNotFoundException;
import java.io.IOException;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.parsers.ParserConfigurationException;

import org.w3c.dom.Document;
import org.w3c.dom.NamedNodeMap;
import org.w3c.dom.Node;
import org.w3c.dom.NodeList;
import org.xml.sax.SAXException;

public class DomTest {

	public void parserXml(String fileName) {
		try {
			DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
			DocumentBuilder db = dbf.newDocumentBuilder();
			Document document = db.parse(fileName);
			NodeList users = document.getChildNodes();

			for (int i = 0; i < users.getLength(); i++) {
				Node user = users.item(i);
				
				
				NodeList userInfo = user.getChildNodes();
				for (int j = 0; j < userInfo.getLength(); j++) {
					Node node = userInfo.item(j);
					NodeList userMeta = node.getChildNodes();
					
					for (int k = 0; k < userMeta.getLength(); k++) {
						if (userMeta.item(k).getNodeName() != "#text")
							System.out
									.println(userMeta.item(k).getNodeName() + ":" + userMeta.item(k).getTextContent());
					}

					System.out.println();
				}
			}

		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (ParserConfigurationException e) {
			e.printStackTrace();
		} catch (SAXException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	public static void main(String[] args) {
		DomTest domj4 = new DomTest();
		// 获取文件路径
		String path = Dom4jTest.class.getClassLoader().getResource("user.xml").getPath();
		System.out.println(path);
		domj4.parserXml(path);
	}
}

```