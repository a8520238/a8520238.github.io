---
title: mybatis
date: 2021-2-28
cover:
top_img:
categories: 项目相关
tags: 
mathjax: true
katex: true
---
# 1 mybatis与JDBC区别

1. Mybatis通过参数映射方式，可以将参数灵活的配置在SQL语句中的配置文件中，避免在Java类中配置参数（JDBC）
2. Mybatis通过输出映射机制，将结果集的检索自动映射成相应的Java对象，避免对结果集手工检索（JDBC）
3. Mybatis可以通过Xml配置文件对数据库连接进行管理。

# 2 验证mybatis的测试代码
```
//使用productMapper.xml配置文件
public class TestClient { //定义会话SqlSession
    SqlSession session =null;

    @Before public void init() throws IOException { //定义mabatis全局配置文件
        String resource = "SqlMapConfig.xml"; //加载mybatis全局配置文件 //InputStream inputStream = TestClient.class.getClassLoader().getResourceAsStream(resource);
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(inputStream); //根据sqlSessionFactory产生会话sqlsession
        session = factory.openSession();    
    } //查询所有user表所有数据
    @Test public void testSelectAllUser() {
        String statement = "com.sl.mapper.ProductMapper.selectAllProduct";
        List listProduct =session.selectList(statement); for(Product product:listProduct)
        {
            System.out.println(product);
        } //关闭会话
        session.close();    
    }
}
```
# 3 Mybatis数据源
Mybatis的数据源分为使用连接池的PooledDataSource和不适用连接吃的UnPooledDataSource

[mybatis架构与原理](https://www.cnblogs.com/chongaizhen/p/11170595.html)

[mybatis配置和使用](https://www.cnblogs.com/yang5201314/p/5945171.html)

[Mybatis数据源](https://blog.csdn.net/luanlouis/article/details/37671851)

[数据库连接池](https://www.cnblogs.com/cocoxu1992/p/11031908.html)

[mybatis代码执行原理与设计模式](https://www.cnblogs.com/toov5/p/11186569.html)