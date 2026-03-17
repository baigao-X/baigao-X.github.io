# 消息机制<no value>

# 1. 消息分类
## 1.1. 标准windows消息

	除了WM_COMMAND 之外以WM_ 开头的消息都是标准消息

## 1.2. 命令消息

	下消息名为WM_COMMAND，消息中附带了标识符ID来区分来自哪个菜单、工具栏按钮或者加速键的消息

## 1.3. 通知消息

一般由子窗口发送给父窗口，消息名也为WM_COMMAND。 附带了空间通知码来区分控件



# 2. 实现消息的组成

## 2.1. 消息映射入口：

在实现文件中由**BEGIN_MESSAG_MAP**和**END_MESSAGE_MAP**之间的内容成为消息映射入口项
``` cpp
BEGIN_MESSAG_MAP

END_MESSAGE_MAP
```
 
## 2.2. 消息宏调用

 **DECLARE_MESSAGE_MAP()**，一般这个宏调用写在类定义的结尾处。

## 2.3. 消息函数声明
以**afx_msg**