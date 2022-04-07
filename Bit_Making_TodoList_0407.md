# Todo-List 만들기 정리

<br>

# 1. **GET (화면 출력)**

![C1C9AFA8-C066-4129-AD07-BEE484883BE5.jpeg](/Image/0408_1.jpg)


### **1-1)**

주소창에 URL(http://192.168.0.00:8080/todo/list)을 입력하면, Browser가 Server에 GET 방식의 request를 보낸다.

### **1-2)**

Server는 Controller에게 /todo/list를 화면에 출력하라는 request를 보낸다.

### **1-3)**

Controller는 **getRequestDispatcher** 메소드를 통해서 화면 출력에 필요한 정보를 request에 담아서 jsp 파일에 보낸다.

```sql
request.getRequestDispatcher("/WEB-INF/todo/list.jsp")
.forward(request, response);
```

- “/WEB-INF/todo/list.jsp”를 Dispatcher(전송하라)
- forward는 행위

### **1-4)**

jsp는 컴파일할 때 자바 코드 out.write를 통해 브라우저로 향하는 outputstream에 텍스트를 보내게 된다. (이때 텍스트 형식은 html) 

Server는 html 형식으로 텍스트를 받아 Browser에 전송하고, 

Browser는 화면에 출력한다.

<br>

---

# 2. **DB 데이터 가져오기**

![8D5D4A96-295D-4B9B-8895-5D17AD202303.jpeg](/Image/0408_2.jpg)

### **2-1)**

주소창에 URL(http://192.168.0.00:8080/todo/list)을 입력하면, Browser가 Server에 GET 방식의 request를 보낸다.

### **2-2)**

Server는 Controller에게 /todo/list 데이터를 화면에 출력하라는 request를 보낸다.

### **2-3)**

Controller는 View에게 list를 가져오라고 요청한다.

(이때, list의 데이터가 없을 경우 Model(DAO)에게 데이터를 가져오라고 요청한다.)

### **2-4)**

Model(DAO)는 DB에 연결하여 SQL 문장을 통해 데이터를 가져온다.

**→ 이때 코드에서 필요한 것!**

```sql
**1. Connection** 
 - Driver가 만들어 주는 것으로, 드라이버 설치 필요 
 - 자주 써주니까 Util 패키지 - enum 클래스의 Instance 로 만들어 준다)

**2. PreparedStatement**
 - executeQuery() 메소드를 통해 결과 데이터를 ResultSet 객체에 담아 가져온다.

**3. ResultSet**
 - next() 메소드를 통해 테이블 한 줄씩 읽어서 List<Todo>로 만든다. (DTO 타입의 List 배열) 
```

[추가 설명]

- **preparedStatement**
    - preparedStatement는 DB에 전달할 SQL문을 작성하고 전송하는 역할을 한다.
        - 문자 그대로 SQL문의 구조를 미리 만들어 두고, 사용자의 입력에 따라 바뀌는 값 등을 ‘?’(물음표)로 비워둔다.
        - 여기서 ‘?’는 일종의 파라미터로 setAtrribute()를 통해 ‘?’에 들어갈 값을 유동적으로 변경할 수 있다. (즉, 사용자가 값을 입력한다면 ‘?’(파라미터) 안에 들어가서, SQL의 일부가 되어 전달된다.)
    - preparedStatement 메소드
        - **executeQuery()→ Resultset** 객체에 결과값을 담아 반환
        - **executeUpdate()** → 수행결과를 **int** 값으로 반환
        - **execute → boolean** 타입으로 반환
    - select는 executeQuery() 메소드를 쓰고 ResultSet이라는 인터페이스를 구현한 클래스 객체를 가져온다.
    - 참고 : [https://mozi.tistory.com/26](https://mozi.tistory.com/26)
        
        [https://docs.microsoft.com/ko-kr/sql/connect/jdbc/reference/sqlserverpreparedstatement-members?view=sql-server-ver15](https://docs.microsoft.com/ko-kr/sql/connect/jdbc/reference/sqlserverpreparedstatement-members?view=sql-server-ver15)
        

- **ResultSet**
    - ResultSet의 next() 메소드를 while문을 통해 테이블 한 줄씩 읽어주는 작업을 반복한다.
    - 끝까지 읽은 후, 결과 데이터를 객체로 담아줘야 하는데, 이때 쓰는 방법은 2가지
        
        1) DTO와 VO 둘다 쓰거나
        
        2) DTO와 VO 둘 중 하나만 쓰기  (→ 수업에선 DTO 하나만 써보는 방식으로 함)
        

### **2-5)**

Model(DAO)는 ResultSet 객체로 결과 데이터를 가져오고, builder를 통해서 Todo 객체로 바꿔 담아준 뒤, 이를 Controller에 전달한다.

### **2-6)**

Controller는 forward를 통해 request를 View(jsp)에 보내준다.

### **2-7)**

View(jsp)는 JSTL + EL 이용해서 (c : forEach todoList 돌리기) Server로 보낸다.

<br>

---

# 코드

- **ConnectionUtil.java**

```sql
package org.zerock.dao;

import java.sql.Connection;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

public enum ConnectionUtil {
	INSTANCE;
	
	private HikariDataSource dataSource;
	
	ConnectionUtil(){
		
		try {
			Class.forName("oracle.jdbc.driver.OracleDriver");
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:oracle:thin:@192.168.0.00:0000:XE");
        config.setUsername("name");
        config.setPassword("password");
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        
        dataSource = new HikariDataSource(config);
	}
	
	public Connection getConnection()throws Exception {
		return dataSource.getConnection();
	}
	
}
```

<br>

- **TodoDAO.java**

```sql
package org.zerock.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.util.ArrayList;
import java.util.List;

import org.zerock.domain.Todo;

import lombok.Cleanup;
import lombok.extern.log4j.Log4j2;

@Log4j2
public enum TodoDAO {
	INSTANCE;
	
	private static final String INSERT = 
			" insert into t_todo (tno, title, dueDate, writer)"
			+ " values (seq_t.nextval, ?, ?, ?)";
	
	
	
	public void insert(Todo todo)throws Exception {
		@Cleanup Connection con = ConnectionUtil.INSTANCE.getConnection();
		@Cleanup PreparedStatement pstmt = con.prepareStatement(INSERT);
		//?에 값을 1부터 시작 
		pstmt.setString(1, todo.getTitle());
		//? 날짜 
		pstmt.setDate(2, java.sql.Date.valueOf(todo.getDueDate()));
		//? 문자
		pstmt.setString(3, todo.getWriter());
		
		//DML int
		int count = pstmt.executeUpdate();
		
		log.info("===========");
		log.info(count);
		
	}

	public List<Todo> getList(int page)throws Exception {
		
	
		String query =
				"select * from"
				+ " (select"
				+ "  rownum rn, tno, title, duedate"
				+ "  from t_todo"
				+ "  where tno >0 and rownum <= (? * 10)"
				+ "  order by tno desc"
				+ ")"
				+ " where rn > ((?-1) * 10)";
		
		@Cleanup Connection con = ConnectionUtil.INSTANCE.getConnection();
		@Cleanup PreparedStatement pstmt = con.prepareStatement(query);
		pstmt.setInt(1, page);
		pstmt.setInt(2, page);
		
		@Cleanup ResultSet rs = pstmt.executeQuery();
		
		List<Todo> result = new ArrayList<>();
		
		while(rs.next()) {
			Todo todo = Todo.builder().tno(rs.getInt(2))
					.title(rs.getString(3))
					.dueDate(rs.getDate(4).toLocalDate()).build();
			
			result.add(todo);
		}//end while
		return result;
	}
}
```
<br>

- **list.jsp**

```sql
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>

<%@include file="../includes/header.jsp" %>

<!-- Page content-->
<div class="container-fluid">
    <h1 class="mt-4">Todo List</h1>

<ul class="list-group">
  <c:forEach items="${todoList}" var="todo">
  <li class="list-group-item">${todo.tno }  --- ${todo.text }</li>
  </c:forEach>
</ul>

</div>

<%@include file="../includes/footer.jsp" %>
```

