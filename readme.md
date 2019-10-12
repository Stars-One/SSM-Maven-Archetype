# SSM-Template
整合了spring+spring mvc+mybatis的maven骨架，内置oracle驱动和MyBatis分页插件
## 引入
下载地址：[ssm-template-archetype.zip](https://files.cnblogs.com/files/kexing/ssm-template-archetype.zip)
解压缩，之后命令行进入到目录下，使用`mvn clean install`进行安装

之后，在IDEA的Maven项目创建界面添加此骨架

![](https://github.com/Stars-One/SSM-Maven-Archetype/blob/master/img/show.png?raw=true)

之后像平常那样创建一个maven项目
## 使用注意事项
### 修改配置
根据自己的要求，进行配置的更改

首先是数据源的配置`db.properties`

**spring-config.xml**	
72行 mapper的路径（相对于resources目录来说）

105行 dao类中的包，

```
<mybatis:scan base-package="dao" />
```
是会自动扫描Mapper所在的包，并生成一个id为xx的bean（Dao对象），之后我们就可以通过spring容器获得这个对象，使用这个对象来进行CRUD操作	

相当于下面的代码
```
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	<property name="basePackage" value="dao" />
	<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>
```

### Mybatis使用
Mybatis中，我们可以按照约定，把接口类和mapper.xml文件对应起来，从而更为简单地实现CRUD操作

项目中，我有这样的对应的接口类和mapper.xml文件

![](https://img2018.cnblogs.com/blog/1210268/201910/1210268-20191011232007566-308662529.png)

UserMapper中的namespace就是为dao.UserDao，这里**namespace一定要对应上接口类UserDao**

**UserMapper.xml**
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
namespace: 命名空间，用于标识每一个Mapper XML文件中的语句，预防在不同的Mapper XML文件中存在相同的语句ID
-->
<mapper namespace="dao.UserDao">
    <!--
        resultType: 也称为自动映射，只有在表的列名与POJO类的属性完全一致时使用，会比较方便，全类名
    -->
    <select id="select" resultType="model.Users">
        select * from USERS
    </select>
</mapper>
```
**UserDao.java**
```
package dao;

import java.util.List;

import model.Users;

/**
 * @author StarsOne
 * @date Create in  2019/10/11 0011 20:22
 * @description
 */
public interface UserDao {
    List<Users> select();
}
```
使用的话就通过这样
```
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring/spring-config.xml");
//spring中自动装载，第一个字母小写，如果是EmployeeDao，则是就是employeeDao
UserDao userMapper = (UserDao) context.getBean("userDao");
List<Users> users = userMapper.selectAll();
for (Users user : users) {
	System.out.println(user.toString());
}
```
### 分页使用
```
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring/spring-config.xml");
AccountDao acountDao = (AccountDao) context.getBean("accountDao");
//查询第一页，每页有两条数据
PageHelper.startPage(1, 2);
List<Account> accounts = acountDao.selectAll();
for (Account account : accounts) {
	System.out.println(account.toString());
}
```
