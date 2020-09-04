# 简介
我们用生成文件，并进行下载
调用下面的方法就能实现
```
// 下载文件方法
function funDownload(content, filename) {
    var eleLink = document.createElement('a');
    eleLink.download = filename;
    eleLink.style.display = 'none';
    // 字符内容转变成blob地址
    var blob = new Blob([content]);
    eleLink.href = URL.createObjectURL(blob);
    // 触发点击
    document.body.appendChild(eleLink);
    eleLink.click();
    // 然后移除
    document.body.removeChild(eleLink);
};
```
原理其实很简单，我们可以将文本或者JS字符串信息借助Blob转换成二进制，然后，作为<a>元素的href属性，配合download属性，实现下载。

