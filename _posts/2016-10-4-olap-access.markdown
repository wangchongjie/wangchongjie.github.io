---
layout:     post
title:      "报表中间件olap-access框架"
subtitle:   " \"通用报表ORM框架.\""
date:       2016-07-05 21:55:00
author:     "WangChongjie"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - ORM框架
---
业界各类用户产品和商业产品都会有大量的报表，而对于OLAP类型的报表，无论其存储介质应用或上层查询模式均与传统的RDBMS
体系有较大区别。Olap-access是报表领域模型的ORM框架，旨在提升用户的开发效率，只需关注报表建模，其余逻辑由底层框架来支持。

## Olap-access介绍

## 1.1 Olap-access是什么
Olap-access是一个报表olap（On-Line Analysis Processing）数据查询中间件，提供olap分库路由、上卷表智能适配、olap数据查询及报表数据ORM等功能。后续版本中，会引入插件化的分布式缓存，进一步提升报表数据的查询性能。

该框架设计初衷是：为OlapEngine提供灵活、易用、高性能、无侵入的报表数据查询方案。更进一步，理论上不局限于OlapEngine，只要报表存储系统支持SQL接口，并遵循一定的规范，均可应用olap-access实现报表数据的查询。

Olap-access可理解为与业务无关的报表DAO层封装，为“olap入口、传送门”的意思。

## 1.2 Olap-access功能概述
1、分库路由
插件化地配置分库路由策略，并提供默认实现，olap-access应用上层对分库规则透明，支持单库或跨库userid（主key id名称可自定义）的olap报表数据查询。

2、数据检索
支持多用户查询、支持多olap表查询，支持key列、values列、扩展列查询，支持分页、排序功能、支持全字段过滤、支持时间粒度（按小时、天、周、月、季度、年）聚合、支持物化视图表的智能适配（时间维度表、业务层级表均可）、支持排序列智能擦除、支持自定义Olap表可查询起始时间。

3、ORM映射
框架提供两种查询模式：
1、List<Obj>返回模式：用户建模后，基于注解可完成olap查询返回对象的自动组装，并提供回调方法afterAssemble，实现用户自定义逻辑处理（如自定义日期格式化等），用户对SQL透明、无感知。
2、List<Map>返回模式：该模式较为粗犷，支持用户自定义SQL，返回结果为List<Map>类型。通常情况下，建议优先使用#1方式。

4、多表查询
多表查询支持两种模式：
1、同一个item对象上建模，框架完成多表查询并完成ORM及字段Join。
2、为每张表的item分别建模，框架完成查询及ORM，用户调用提供的util方法，完成List<obj>的Join。

5、插件化缓存
提供插件化缓存的接口，用户实现该接口（NoSql，用户可自由选型）后，即可实现olap数据的查询缓存，提升Olap数据的查询性能。

## 1.3 Olap-access产生背景
自从2008年北斗1.0以来，伴随着业务需求的不断增加，报告模块趋于膨胀，报告模块代码量已经达到了20万行的量级，由于代码量及业务的增长，出现很多问题，包括：

```xml
	新增业务维度效率低，升级逻辑复杂
	代码冗余现象
	报表查询接口不够简单易用
	List<Map>方式的内存、及处理开销的浪费
	系统可扩展性差，性能受到限制
```

# 2 Olap-access应用示例

Olap-access框架的报表建模示例如下：

```java
@OlapTable(
	name=Constants.TABLE.GROUP, 
	keyVal= {Constants.COLUMN.PLANID, Constants.COLUMN.GROUPID}, 
	basicVal = {Constants.COLUMN.SRCHS, Constants.COLUMN.CLKS},
	extCol = {Constants.COLUMN.CTR}, 
	extExpr = {Constants.EXPR.CTR}
)
public class GroupViewItem extends BaseItem {	

	@OlapColumn(Constants.COLUMN.PLANID)
	private Integer planId;
	
	@OlapColumn(Constants.COLUMN.GROUPID)
	private Integer groupId;
	
	@OlapColumn(Constants.COLUMN.SRCHS)
	private long srchs;//展现
	
	@OlapColumn(Constants.COLUMN.CLKS)
	private long clks;//点击
	
	@OlapColumn(Constants.COLUMN.CTR)
	private BigDecimal ctr;//点击率
	
	private Sting planName;//计划名称
	private Sting groupName;//推广组名称

	@Override
	public void afterAssemble(int timeUnit){
		super.afterAssemble(timeUnit);
		// DO OTHER THINGS
	}
}

// GETTER & SETTER 方法略
```

Olap-access的业务开发应用示例如下：

```java
@Service
public class GroupStatServiceImpl extends AbstractOlapService implements GroupStatService {

@Override
public List<GroupViewItem> queryGroupData(int userId,List<Integer> planIds, List<Integer> groupIds, Date from, Date to,String column, int order, int timeUnit) {

List<String> filters = new LinkedList<String>();
OlapUtils.makeNumberFilter(filters,Constants.COLUMN.PLANID,planIds);
OlapUtils.makeNumberFilter(filters,Constants.COLUMN.GROUPID,groupIds;

ReportRequest<GroupViewItem> rr = new ReportRequestBuilder<GroupViewItem>(){}
				.setUserId(userId)
				.setFrom(from)
				.setTo(to)
				.setTimeUnit(timeUnit)
				.setColumn(column)
				.setOrder(SortOrder.val(order))
				.setFilters(filters)
				.build();

		return super.getStorageData(rr);
	}
}
```

由示例可见，用户只需对报表领域模型建模，进行@OlapTable和@OlapColumn注解标注。便可直接使用可扩展的Olap
查询接口，完成报表数据的ORM映射、路由、检索、缓存等处理，极大地提升了报表的开发效率。