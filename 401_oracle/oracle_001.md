# JDBC连接ORACLE的三种URL格式



使用jdbc连接oracle时url有三种格式

# 格式一: Oracle JDBC Thin using an SID

```
jdbc:oracle:thin:@host:port:SID 
例如: jdbc:oracle:thin:@localhost:1521:orcl 
```

这种格式是最简单也是用得最多的。

你的oracle的sid可以通过一下指令获得： 

```
sqlplus / as sysdba 
select value from v$parameter where name='instance_name';
```

测试代码：

```
import java.sql.*;

public class TestOrclConnect {

    public static void main(String[] args) {
        ResultSet rs = null;
        Statement stmt = null;
        Connection conn = null;
        try {
            Class.forName("oracle.jdbc.driver.OracleDriver");
            String dbURL = "jdbc:oracle:thin:@localhost:1521:orcl";
            conn = DriverManager.getConnection(dbURL, "admin2", "123");
            System.out.println("连接成功");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                if (rs != null) {
                    rs.close();
                    rs = null;
                }
                if (stmt != null) {
                    stmt.close();
                    stmt = null;
                }
                if (conn != null) {
                    conn.close();
                    conn = null;
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}

```


# 格式二: Oracle JDBC Thin using a ServiceName

```
jdbc:oracle:thin:@//host:port/service_name 
例如: jdbc:oracle:thin:@//localhost:1521/orcl.city.com 
```

注意这里的格式，**@后面有//, port后面:换成了/**,这种格式是Oracle 推荐的格式，因为对于集群来说，每个节点的SID 是不一样的，但是SERVICE_NAME 确可以包含所有节点。 

你的oracle的service_name可以通过以下方式获得： 

```
sqlplus / as sysdba 
select value from v$parameter where name='service_names';
```

测试代码：

```
import java.sql.*;

public class TestOrclConnect {

    public static void main(String[] args) {
        ResultSet rs = null;
        Statement stmt = null;
        Connection conn = null;
        try {
            Class.forName("oracle.jdbc.driver.OracleDriver");
            String dbURL = "jdbc:oracle:thin:@//localhost:1521/orcl.city.com";
            conn = DriverManager.getConnection(dbURL, "admin2", "123");
            System.out.println("连接成功");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            try {
                if (rs != null) {
                    rs.close();
                    rs = null;
                }
                if (stmt != null) {
                    stmt.close();
                    stmt = null;
                }
                if (conn != null) {
                    conn.close();
                    conn = null;
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
}
```


# 格式三：Oracle JDBC Thin using a TNSName

```
jdbc:oracle:thin:@TNSName 
例如:  jdbc:oracle:thin:@TNS_ALIAS_NAME 
```

我在谷歌上找了一些资源，要实现这种连接方式首先要建立tnsnames.ora文件，然后通过System.setProperty指明这个文件路径。再通过上面URL中的@符号指定文件中的要使用到的资源。 
这种格式我现在水平几乎没见过，对于我来说用得到这种的情况并不多吧。当然既然是通过配置文件来读取指定资源肯定也可以直接将资源拿出来放在URL中，直接放在URL中的URL模版是下面这样的（tnsnames.ora这个文件中放的就是@符号后面的那一段代码，当然用文件的好处就是可以配置多个，便于管理）：

配置代码：

```
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.16.91)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=orcl)))
```

测试代码：

```
import java.sql.*;

public class TestOrclConnect {
	public static void main(String[] args) {
	    ResultSet rs = null;
	    Statement stmt = null;
	    Connection conn = null;
	    try {
	        Class.forName("oracle.jdbc.driver.OracleDriver");
	         String dbURL =
	        "jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=localhost)(PORT=1521)))"
	        + "(CONNECT_DATA=(SERVICE_NAME=orcl.city.com)))";
	        conn = DriverManager.getConnection(dbURL, "admin2", "123");
	        System.out.println("连接成功");
	    } catch (ClassNotFoundException e) {
	        e.printStackTrace();
	    } catch (SQLException e) {
	        e.printStackTrace();
	    } finally {
	        try {
	            if (rs != null) {
	                rs.close();
	                rs = null;
	            }
	            if (stmt != null) {
	                stmt.close();
	                stmt = null;
	            }
	            if (conn != null) {
	                conn.close();
	                conn = null;
	            }
	        } catch (SQLException e) {
	            e.printStackTrace();
	        }
	    }
	}
}
```




<br><br><br><br>
原文：https://blog.csdn.net/u012062455/article/details/52442838





