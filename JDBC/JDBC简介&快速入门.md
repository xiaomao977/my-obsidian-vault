![[Pasted image 20251230155257.png]]
```java
//8.x驱动会自动注册，通常不用再Class.forName()  
//1、注册驱动  
//Class.forName("com.mysql.cj.jdbc.Driver");  
  
//2、获取连接  
String url = "jdbc:mysql://127.0.0.1:3306/test";  
String username = "root";  
String password = "xrp1329273428";  
Connection conn = DriverManager.getConnection(url, username, password);  
  
//3、定义SQL语句  
String sql = "update user set age = 58 where id = 1";  
  
//4、获取执行SQL对象  
Statement stmt = conn.createStatement();  
  
//5、执行SQL  
int count = stmt.executeUpdate(sql);//这里返回的影响的行数  
  
//6、处理返回结果  
System.out.println(count);  
  
//7、释放资源  
stmt.close();  
conn.close();
```
