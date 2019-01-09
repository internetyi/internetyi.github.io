---
layout: post
title: spring-boot 数据库事务使用指南
excerpt: 简单事务 使用@Transactional 注解 注意：@Transactional 只能用于public方法，其他方法不生效！
categories: [后端开发]
tags: [java]
---

#### 简单事务
使用`@Transactional` 注解 

> 注意：`@Transactional` 只能用于public方法，其他方法不生效！

``` java
//捕获所有异常并回滚
@Transactional

//指定异常回滚
@Transactional(rollbackFor = {IllegalArgumentException.class}) 

//指定异常不会滚，其他异常回滚
@Transactional(noRollbackFor = {IllegalArgumentException.class})


@EnableTransactionManagement //如果mybatis中service实现类中加入事务注解，需要此处添加该注解


@Transactional(propagation = Propagation.REQUIRED,isolation = Isolation.DEFAULT,timeout=36000,rollbackFor=Exception.class)
```
#### 复杂事务
手动指定事务commit 、rollback

``` java
#注入事务管理bean
@Autowired
private PlatformTransactionManager transactionManager;

#显示开启事务
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

#业务控制是否回滚
try {
    TradeQuicklyBatchBody tradeInfo = saveBugAlbumData(tradNo, albumInfoList, request);
    updateStatusParam.setPayTradeNo(tradeInfo.getPayTradeNo());
    updateStatusParam.setStatus(1);
	#正常情况commit
    transactionManager.commit(status);
} catch (BaseException e) {
    log.warn("购买专辑保存数据失败, {}", e.toString());
    updateStatusParam.setStatus(2);
	#异常情况rollback
    transactionManager.rollback(status);
}
```
其他高级特性自行查看spring 手册