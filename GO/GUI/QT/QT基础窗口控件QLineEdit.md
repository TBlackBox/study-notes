# 简介

QLineEdit类是一个单行文本控件，可输入单行字符串，可以设置回显模式(Echomode)和掩码模式
1. 回显模式(Echomode)
回显模式就是当键盘被按下后，显示了什么

- Normal 正常的回显模式
- NoEcho 不回显模式(什么都没出现)
- Password 密码
- PasswordEchoOnEdit 先是显示，然后过了几秒就不显示


2. 校验器
比如只能限制输入整数或满足一定条件的字符串

可以创建正则的检验器，然后设置。

3. 掩码
要限制用户的输入，除了刚刚的校验器外，还有掩码，比如在修改IP时，这个就是掩码
```
A    ASCII字母字符是必须输入的(A-Z、a-z)
a    ASCII字母字符是允许输入的,但不是必需的(A-Z、a-z)
N    ASCII字母字符是必须输入的(A-Z、a-z、0-9)
n    ASII字母字符是允许输入的,但不是必需的(A-Z、a-z、0-9)
X    任何字符都是必须输入的
x    任何字符都是允许输入的,但不是必需的
9    ASCII数字字符是必须输入的(0-9)
0    ASCII数字字符是允许输入的,但不是必需的(0-9)
D    ASCII数字字符是必须输入的(1-9)
d    ASCII数字字符是允许输入的,但不是必需的(1-9)
#    ASCI数字字符或加减符号是允许输入的,但不是必需的
H    十六进制格式字符是必须输入的(A-F、a-f、0-9)
h    十六进制格式字符是允许输入的,但不是必需的(A-F、a-f、0-9)
B    二进制格式字符是必须输入的(0,1)
b    二进制格式字符是允许输入的,但不是必需的(0,1)
>    所有的字母字符都大写
<    所有的字母字符都小写
!    关闭大小写转换
\    使用"\"转义上面列出的字符
```