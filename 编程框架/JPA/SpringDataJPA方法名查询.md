  * 方法名的约定：
     *      findBy : 查询
     *            对象中的属性名（首字母大写） ： 查询的条件
     *            CustName
     *            * 默认情况 ： 使用 等于的方式查询
     *                  特殊的查询方式
     *
     *  findByCustName   --   根据客户名称查询
     *
     *  再springdataJpa的运行阶段
     *          会根据方法名称进行解析  findBy    from  xxx(实体类)
     *                                      属性名称      where  custName =
     *
     *      1.findBy  + 属性名称 （根据属性名称进行完成匹配的查询=）
     *      2.findBy  + 属性名称 + “查询方式（Like | isnull）”
     *          findByCustNameLike
     *      3.多条件查询
     *          findBy + 属性名 + “查询方式”   + “多条件的连接符（and|or）”  + 属性名 + “查询方式”