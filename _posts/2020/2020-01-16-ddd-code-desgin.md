---
title:  基于 DDD 的代码设计
author:
  name: superhsc
  link: https://github.com/maxpixelton
date: 2020-01-16 23:33:00 +0800
categories: [系统设计, 领域驱动设计 DDD]
tags:  [DDD]
math: true
mermaid: true
---

## 整个演示代码的架构

![This is a image](https://maxpixelton.github.io/images/assert/ddd/1601.png)

如上图：

- demo-ddd-trade：基于 DDD 的单体应用与一个完整的微服务应用；
- demo-parent：早期框架设计的
  - demo-service-config：微服务配置中心 config
  - demo-service-eureka: 微服务注册中心 eureka
  - demo-service-turbine：各微服务断路器监控 turbine
  - demo-service-zuul：服务网关 zuul
  - demo-service-parent：各业务微服务（五数据库访问）的服务项目
    - demo-service-support：各业务微服务（无数据库访问）底层技术框架
    - demo-service-customer：用户管理微服务（无数据库访问）
    - demo-service-product：产品管理微服务（无数据库访问）
    - demo-service-supplier：供应商管理微服务（无数据库访问）
- demo-ddd-parent：是基于 DDD 设计的微服务设计
  - demo-service-config：微服务配置中心 config
  - demo-service-eureka: 微服务注册中心 eureka
  - demo-service-turbine：各微服务断路器监控 turbine
  - demo-service-zuul：服务网关 zuul
  - demo-ddd-service-parent：各业务微服务（有数据库访问）的父项目
    - demo-ddd-service-support：各业务微服务底层技术框架（有数据库访问）
    - demo-ddd-service-customer：用户管理微服务（有数据库访问）
    - demo-ddd-service-product：产品管理微服务（有数据库访问）
    - demo-ddd-service-supplier：供应商管理微服务（有数据库访问）
    - demo-ddd-service-order：订单管理微服务（有数据库访问）



## 单 Controller 设计实现

与以往不同，在整个系统中只有几个 Controller，并下沉到了底层技术中台 `demo-ddd-service-support`，它们包括以下几部分：

- `OrmController`：用于增删改操作，以及基于 key 值的 load、get 操作，它们通常基于 `DDD` 进行设计；
- `QueryController`：用于基于 `SQL` 语句形成的查询分析报表，它们通常不基于 `DDD` 进行设计，但查询结果会形成领域对象，并基于 `DDD` 进行数据补填；
- 其他 Controller，用于如 `ExcelController` 等特殊的操作，是继承以上两个类的功能扩展。

`OrmController` 接收诸如 `orm/{bean}/{method}` 的请求，bean 是配置在 Spring 中的 bean，method 是 bean 中要调用的方法。由于这是一个基础框架，没有限定前端可以调用哪些方法，因此实际项目需要在此之上增加权限校验。该方法既可以接收 GET 方法，也可以接收 POST 方法，因此其他的参数可以根据 GET/POST 各自的方式进行传递。

这里的 bean 对应的是后台的 Service。Service 的编写要求所有的方法，如果需要使用领域对象必须放在第一个参数上。如果第一个参数是简单的数字、字符串、日期等类型，就不是领域对象，否则就作为领域对象，依次从前端上传的 JSON 中获取相应的数据予以填充。这里暂时不支持集合，也不支持具有继承关系的领域对象，待我日后完善。判定代码如下：

```java
/**
  * check a parameter whether is a value object.
  * @param clazz
  * @return yes or no
  * @throws IllegalAccessException 
  * @throws InstantiationException 
  */
private boolean isValueObject(Class<?> clazz) {
    if(clazz==null) {
        return false;
    }
    
    if(clazz.equals(long.class) || clazz.equals(int.class) || clazz.equals(double.class) || clazz.equals(float.class) || clazz.equals(short.class)) {
        return false;
    }
    if(clazz.isInterface()) { 
        return false;
    }
    if(Number.class.isAssignableFrom(clazz)) {
        return false;
    }
    if(String.class.isAssignableFrom(clazz)) {
        return false;
    }
    if(Date.class.isAssignableFrom(clazz)) {
        return false;
    }
    if(Collection.class.isAssignableFrom(clazz)) {
        return false;
    }
    return true;
}
```

这里的开发规范除了要求 Service 的所有方法中的领域对象放第一个参数，还要求前端的 JSON 与领域对象中的属性一致，这样才能完成自动转换，而不需要为每个模块编写 Controller。

`QueryController` 接收诸如` query/{bean}` 的请求，这里的 bean 依然是 Spring 中配置的bean。同样，该方法也是既可以接收 GET 方法，也可以接收 POST 方法，并用各自的方式传递查询所需的参数。

如果该查询需要分页，那么在传递查询参数以外，还要传递 page（第几页）与 size（每页多少条记录）。第一次查询时，除了分页，还会计算 count 并返回前端。这样，在下次分页查询时，将 count 也作为参数传递，将不再计算 count，从而提升查询效率。此外，这里还将提供求和功能，敬请期待。



### 单 Dao 的设计实现

以往系统设计的硬伤在于一头一尾：Controller 与 Dao。它既要为每个模块编写大量代码，也使得系统设计非常不 DDD，令日后的变更维护成本巨大。

因此，在大量系统设计问题分析的基础上，提出了单 Controller 与单 Dao 的设计思路。前面说了单 Controller 的设计，现在来看一看单 Dao 的设计。

诚然，当今主流是使用注解。然而，注解的使用存在诸多的问题。

- 首先，它会带来业务代码与技术框架的依赖，因此当在 Service 中加入注解时，就不得不与 Spring、Springcloud 耦合，使得日后转型其他技术框架困难重重。
- 此外，注解往往适用于一对一、多对一的场景，而一对多、多对多的场景往往非常麻烦。而本框架存在大量一对多、多对多的场景，因此我建议你还是回归到 XML 的配置方式。

在项目中的所有 Service 都要有一个 BasicDao 的属性变量，例如：

```java
public class CustomerServiceImpl implements CustomerService {

	private BasicDao dao;
	/**
	 * @return the dao
	 */
	public BasicDao getDao() {
		return dao;
	}
	/**
	 * @param dao the dao to set
	 */
	public void setDao(BasicDao dao) {
		this.dao = dao;
	}

}

```

接着，在 applicationContext-orm.xml 中，配置业务操作的 Service：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:jee="http://www.springframework.org/schema/jee" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd 
	http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.0.xsd 
	http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd 
	http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd"
	>
	<description>The application context for orm</description>
	<bean id="customer" class="cn.happymaya.ddp.customer.service.impl.CustomerServiceImpl">
		<property name="dao" ref="repositoryWithCache"/>
	</bean>
</beans>
```

这里可以看到，每个 Service 都要注入 Dao，但可以根据需求注入不同的 Dao。

- 如果该 Service 是纯贫血模型，那么注入 BasicDao 就可以了。
- 如果采用了充血模型，包含了一些聚合的操作，那么注入 repository 从而实现仓库与工厂的功能。
- 但如果还希望该仓库与工厂能提供缓存的功能，那么就注入 repositoryWithCache。

例如，在以上案例中：

- SupplierService 实现的是非常简单的功能，注入 BasicDao 就可以了；
- OrderService 实现了订单与明细的聚合，但数据量大不适合使用缓存，所以注入 repository；
- CustomerService 实现了用户与地址的聚合，并且需要缓存，所以注入 repositoryWithCache；
- ProductService 虽然没有聚合，但在查询产品时需要补填供应商，因此也注入repositoryWithCache。

这里需要注意，是否使用缓存，也可以在日后的运维过程中，让运维人员通过修改配置去决定，从而提高系统的可维护性。

完成配置以后，核心是**将领域建模映射成程序设计的模型**。开发人员首先编写各个领域对象。譬如，产品要关联供应商，那么在增加 supplier_id 的同时，还要增加一个 Supplier 的属性：

```java
public class Product extends Entity<Long> {
	private static final long serialVersionUID = 7149822235159719740L;
	private Long id;
	private String name;
	private Double price;
	private String unit;
	private Long supplier_id;
	private Long classify_id;
	private String image;
	private Double original_price;
	private String tip;
	private Supplier supplier;
}
```

注意，在本框架中的每个领域对象都必须要实现 Entity 这个接口，系统才知道你的主键是哪个。

接着，配置 vObj.xml，将领域对象与数据库对应起来：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<vobjs>
  <vo class="cn.happymaya.ddp.product.entity.Product" tableName="Product">
    <property name="id" column="id" isPrimaryKey="true"/>
    <property name="name" column="name"/>
    <property name="price" column="price"/>
    <property name="unit" column="unit"/>
    <property name="classify_id" column="classify_id"/>
    <property name="supplier_id" column="supplier_id"/>
    <property name="image" column="image"/>
    <property name="original_price" column="original_price"/>
    <property name="tip" column="tip"/>
    <join name="classify" joinKey="classify_id" joinType="manyToOne"
          class="cn.happymaya.ddp.product.entity.Classify"/>
    <ref name="supplier" refKey="supplier_id" refType="manyToOne" 
      bean="cn.happymaya.ddp.product.service.SupplierService"
         method="loadSupplier" listMethod="loadSuppliers"/>
  </vo>
  <vo class="cn.happymaya.ddp.product.entity.Classify" tableName="Classify">
  	<property name="id" column="id" isPrimaryKey="true"/>
    <property name="name" column="name"/>
    <property name="parent_id" column="parent_id"/>
    <property name="layer" column="layer"/>
  </vo>
</vobjs>
```

注意，在这里，所有用到 join 或 ref 标签的领域对象，其 Service 都必须使用 repository 或repositoryWithCache，以实现数据的自动补填，或者有聚合的地方实现聚合的操作，而注入 BasicDao 是无法实现这些操作的。

此外，各属性中的 name 配置的是该领域对象私有属性变量的名字，而不是 GET 方法的名字。例如，OrderItem 中配置的是 product_id，而不是 productId，并且该名字必须与数据库字段一致（这是 MyBatis 的要求，我也很无奈）。

有了以上的配置，就可以轻松实现 Service 对数据库的操作，以及 DDD 中那些烦琐的缓存、仓库、工厂、聚合、补填等操作。通过底层技术中台的封装，上层业务开发人员就可以专注于业务理解、领域建模，以及基于领域模型的业务开发，让 DDD 能更好、更快、风险更低地落地到实际项目中。
