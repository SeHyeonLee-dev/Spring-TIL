# JPA 없이 직접 쿼리를 관리할 때의 문제점 정리

## 1. 프로젝트 설정
- JDBC를 사용하여 MySQL과 직접 통신.
- `build.gradle`에 `mysql-connector-java` 추가:
    ```groovy
    implementation 'mysql:mysql-connector-java'
    ```
- `users` 테이블 생성:
    ```sql
    create table users
    (
        id    int auto_increment primary key,
        name  varchar(50)  not null,
        email varchar(100) not null
    );
    ```

---

## 2. JDBC를 활용한 회원 관리 코드
JDBC를 사용하여 직접 데이터를 삽입, 조회, 수정, 삭제하는 DAO(Data Access Object) 클래스.

### 메인 실행 코드
```java
import java.sql.SQLException;

public class RawJdbcMain {
    public static void main(String[] args) {
        UserDAO userDAO = new UserDAO();

        try {
            // 데이터 삽입
            userDAO.insertUser("홍길동", "hong@example.com");
            System.out.println("데이터가 삽입되었습니다.");

            // 데이터 조회
            userDAO.getUsers();

            // 데이터 수정
            userDAO.updateUserEmail("홍길동", "hong_gil_dong@example.com");
            System.out.println("데이터가 수정되었습니다.");

            // 데이터 삭제
            userDAO.deleteUser("홍길동");
            System.out.println("데이터가 삭제되었습니다.");
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

---

### DAO 코드 (UserDAO)
```java
import java.sql.*;

public class UserDAO {
    private final String url = "jdbc:mysql://localhost:3306/jeff_db";
    private final String user = "jeff";
    private final String password = "123123..";

    public void insertUser(String name, String email) throws SQLException {
        try (Connection connection = DriverManager.getConnection(url, user, password);
             PreparedStatement preparedStatement = connection.prepareStatement(
                     "INSERT INTO users (name, email) VALUES (?, ?)") {
            preparedStatement.setString(1, name);
            preparedStatement.setString(2, email);
            preparedStatement.executeUpdate();
        }
    }

    public void getUsers() throws SQLException {
        try (Connection connection = DriverManager.getConnection(url, user, password);
             PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM users")) {
            ResultSet resultSet = preparedStatement.executeQuery();
            while (resultSet.next()) {
                int id = resultSet.getInt("id");
                String name = resultSet.getString("name");
                String email = resultSet.getString("email");
                System.out.println("ID: " + id + ", 이름: " + name + ", 이메일: " + email);
            }
        }
    }

    public void updateUserEmail(String name, String newEmail) throws SQLException {
        try (Connection connection = DriverManager.getConnection(url, user, password);
             PreparedStatement preparedStatement = connection.prepareStatement(
                     "UPDATE users SET email = ? WHERE name = ?")) {
            preparedStatement.setString(1, newEmail);
            preparedStatement.setString(2, name);
            preparedStatement.executeUpdate();
        }
    }

    public void deleteUser(String name) throws SQLException {
        try (Connection connection = DriverManager.getConnection(url, user, password);
             PreparedStatement preparedStatement = connection.prepareStatement(
                     "DELETE FROM users WHERE name = ?")) {
            preparedStatement.setString(1, name);
            preparedStatement.executeUpdate();
        }
    }
}
```

---

## 3. 문제점 분석

### 1) 반복되는 코드
같은 패턴의 코드가 반복되며, 쿼리 메서드가 추가될 때마다 비슷한 코드가 계속 생성됨.

```java
public void getUserById(Long id) throws SQLException {
    try (Connection connection = DriverManager.getConnection(url, user, password);
         PreparedStatement preparedStatement = connection.prepareStatement(
                 "SELECT * FROM users WHERE id = ?")) {
        preparedStatement.setLong(1, id);
        ResultSet resultSet = preparedStatement.executeQuery();
        if (resultSet.next()) {
            System.out.println("ID: " + resultSet.getLong("id") +
                    ", 이름: " + resultSet.getString("name") +
                    ", 이메일: " + resultSet.getString("email"));
        }
    }
}
```
쿼리가 늘어날수록 유지보수 어려움 증가.

---

### 2) SQL에 대한 강한 의존성
`User` 클래스를 도입해도 SQL을 직접 작성해야 하는 문제는 남아 있음.

#### User 클래스 추가
```java
public class User {
    private Long id;
    private String name;
    private String email;

    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // Getter & Setter
}
```

컬럼 추가 시 `User` 클래스와 DAO 내 SQL을 수정해야 함.

---

### 3) 관계형 데이터 모델 처리의 복잡성
새로운 테이블 추가 시 코드 수정이 많아짐.

#### department 테이블 추가
```sql
create table department (
    id int auto_increment primary key,
    id_user int not null,
    department_name varchar(255),
    constraint fk_user_department foreign key (id_user) references users(id) on delete cascade
);
```

#### Department 클래스 추가
```java
public class Department {
    private Long id;
    private Long userId;
    private String departmentName;
}
```

관계형 데이터를 다룰 때 쿼리를 계속 추가해야 하며, 쿼리 수정이 많아질수록 유지보수 부담이 증가.

---

## 4. 결론: JPA 없이 직접 SQL을 다루는 것은 비효율적
- 반복적인 코드 증가로 유지보수 어려움.
- SQL 의존성이 강해 요구사항 변경 시 수정 부담 증가.
- 객체지향적인 설계가 어려워지고, 관계형 데이터 모델을 다루는 것이 복잡해짐.
- JPA를 사용하면 쿼리에 대한 의존성을 제거하고 객체지향적으로 개발 가능.

