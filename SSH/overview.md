# 部署Spring开发环境
## Struts集成Hibernate
- `Struts`框架将表示层和业务层相分离
- `Hibernate`框架提供了灵活的持久层支持

> 在持久层引入**泛型实现机制**可以对代码进行复用，避免重复开发，节约开发成本

集合后的系统分为 3 层：表示层、业务层和持久层
1. **表示层**<br>
 表示层应遵守以下原则：
 + `尽量使用标签`
 + `避免视图层处理对数据库的访问操作`
2. **业务层**<br>
3. **持久层**<br>
 + 常采用`工厂模式`和`DAO模式`来降低应用的业务逻辑和数据库的访问逻辑之间的关联
 + 为了提高代码的可重用性和可维护性，在DAO模式的实现中采用了泛型机制，其具体实现方式如下：
 	- 建立通用的DAO接口类：`public interface GenericDAO<T, ID extends Serializable>`
 	- 建立通用的DAO接口的实现类：`public abstract class GenericHibernateDAO<T, ID extends Serializable> implements GenericDAO<T, ID>`，该类实现了`GenericDAO`接口中定义的各个方法
 	- 通过继承`GenericDAO`接口建立一个具体对象的DAO接口类
 	- 通过继承`GenericHibernateDAO`并实现`PersonDAO`来建立一个具体对象的DAO实现类

### 中文编码和国际化

## Spring集成环境

---

# Spring集成Hibernate

## 在Spring中配置SessionFactory

## 使用HibernateTemplate进行数据库访问
`HibernateTemplate`中提供了3个构造方法
+ `HibernateTemplate()`
+ `HibernateTemplate(org.hibernate.SessionFactory sessionFactory)`
+ `HibernateTemplate(org.hibernate.SessionFactory sessionFactory, boolean allowCreate)`
