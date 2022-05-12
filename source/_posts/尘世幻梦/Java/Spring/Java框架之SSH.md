---
title: Java框架之SSH
categories:
  - Java
  - SSH
music:
  server: netease
  type: song
  id: 440403990
abbrlink: 6d89a949
---

Java 框架之 SSH 整理，之前有过把 SSH（Spring、Spring MVC、Hibernate） 的项目重构为 SSM 的经历，把一些东西整理一下，做个记录。

<!-- more -->

<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=4 orderedList=false} -->

## SSH

在 Hibernate3 中，引入了 Restrictions 类作为 Expression 的替代，以后的版本，不再推荐使用 Expression。

```java{.line-numbers}
// 推荐使用
Criteria cr = session.createCriteria(ChargeRecord.class).add(Restrictions.eq("chargeStatus", "01")).setProjection(Projections.sum("chargeAmount"));
// 不再推荐
TbItemParamExample example = new TbItemParamExample();
Criteria criteria = example.createCriteria();
criteria.andItemCatIdEqualTo(cid);
Criteria ct= session.createCriteria(TUser.class);
//Criteria中可以增加查询条件
ct.add(Expression.eq("name","Erica"));
ct.add(Expression.eq("sex",new Integer(1)));
//Criteria中增加的查询条件可以由表达式对象创建
```
