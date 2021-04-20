# 简介

http://c.biancheng.net/view/1854.html

[Qt](http://c.biancheng.net/qt/) 中用于项（Item）处理的组件有两类：

- 一类是 Item Views，包括 QListView、QTreeView、 QTableView、QColumnView 等；
  
> Item Views 基于模型/视图（Model/View)结构，视图（View)与模型数据（Model Data)关联实现数据的显示和编辑

- 一类是 Item Widgets，包括 QListWidget、QTreeWidget 和 QTable Widget。

> 直接将数据存储在每一个项里，例如，QListWidget 的每一行是一个项，QTreeWidget 的每个节点是一个项，QTableWidget 的每一个单元格是一个项。一个项存储了文字、文字的格式、自定义数据等。

  



# Item Widgets

Item Widgets 是 GUI 设计中常用的组件