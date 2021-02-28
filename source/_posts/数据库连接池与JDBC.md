---
title: 数据库连接池与JDBC
date: 2021-2-28
cover:
top_img:
categories: 项目相关
tags: 
mathjax: true
katex: true
---
# 1 为什么要使用数据库连接池

1. 数据库、http、netty连接池、jdk线程池，本质上都是连接池技术
2. 连接池技术核心：通过减少对于连接创建、关闭来提升性能。用于用户后续使用，好处是后续使用不用在创建连接以及线程，因为这些都需要相关很多文件、连接资源、操作系统内核资源支持来完成构建，会消耗大量资源，并且创建、关闭会消耗应用程序大量性能。
3. 网络连接本身会消耗大量内核资源，在linux系统下，网络连接创建本身tcp/ip协议栈在内核里面，连接创建关闭会消耗大量文件句柄（linux中万物皆文件，一种厉害抽象手段）系统资源。当下更多是应用tcp技术完成网络传输，反复打开关闭，需要操作系统维护大量tcp协议栈状态。

- 用户第一次登录需要建立连接，之后即可复用
# 2 使用JDBC连接池的好处

1. 连接复用。通过建立一个数据库连接池以及一套连接使用管理策略，使得一个数据库连接可以得到高效、安全的复用，避免了数据库连接频繁建立、关闭的开销。
2. 对于共享资源，有一个很著名的设计模式：资源池。该模式正是为了解决资源频繁分配、释放所造成的问题的。把该模式应用到数据库连接管理领域，就是建立一个数据库连接池，提供一套高效的连接分配、使用策略，最终目标是实现连接的高效、安全的复用。

# 3 JDBC编程的六个步骤

1. 加载驱动程序：`Class.forname("com.mysql.cj.jdbc.Driver");`
2. 建立连接：`Connection connection=DriverManager.getConnection("jdbc:mysql://localhost/Contacts?serverTimezone=UTC","root", "Cc229654512");`
3. 创建语句：`Statement statement=connection.createStatement();`
4. 执行语句: `Result SetresultSet=statement.executeQuery("select Name, PhoneNumber, Email, QQ,Note from Contacts");`
5. 处理ResultSet
```
while(resultSet.next())

{
      System.out.println(resultSet.getString(1)+"\t"+resultSet.getString(2)+"\t"+resultSet.getString(3));
}
```
6. 关闭连接
```
 rs.close();
 state.close();
 conn.close();
```
7. 举例
```
1.  public  void test_insert()  

2.  {  

3.      String driver="oracle.jdbc.driver.OracleDriver";  

4.      String url="jdbc:oracle:thin:@127.0.0.1:1521:orcl";//orcl为sid  

5.      String user="briup";  

6.      String password="briup";  

7.      Connection conn=null;  

8.       Statement stat=null;  

9.      try {  

10.         //1、注册驱动  

11.         Class.forName(driver);  

12.         //2、获取连接  

13.          conn= DriverManager.getConnection(url, user, password);  

14.          //System.out.println(conn);  

15.         //3、创建statement对象  

16.         stat=conn.createStatement();  

17.          //4、执行sql语句  

18.          String sql="insert into lover values(5,'suxingxing',to_date('21-9-2016','dd-mm-yyyy'))";  

19.          stat.execute(sql);  

20.          //System.out.println(stat.execute(sql));  

21.          //5、处理结果集,如果有的话就处理，没有就不用处理，当然insert语句就不用处理了  

22.     } catch (Exception e) {  

23.         e.printStackTrace();  

24.     }  

25.     finally{  

26.         //6、关闭资源  

27.         try {  

28.             if(stat!=null)stat.close();  

29.         } catch (SQLException e) {  

30.             e.printStackTrace();  

31.         }  

32.         try {  

33.             if(conn!=null)conn.close();  

34.         } catch (SQLException e) {  

35.             e.printStackTrace();  

36.         }  

37.     }  

38. }  
```