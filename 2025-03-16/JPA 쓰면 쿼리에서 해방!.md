# JDBC vs JPA: 무엇이 더 좋은가?

## 1. JDBC와 JPA 개요
### JDBC (Java Database Connectivity)
- 자바에서 데이터베이스와 직접 연결하여 SQL을 실행하는 API
- `Connection`, `Statement`, `ResultSet` 등의 객체를 사용하여 데이터를 조회 및 조작
- SQL을 직접 작성해야 하므로 SQL 문법에 대한 이해가 필요

### JPA (Java Persistence API)
- 자바 객체를 데이터베이스의 테이블과 매핑하여 관리하는 ORM (Object-Relational Mapping) 기술
- Hibernate, EclipseLink, OpenJPA 등의 구현체가 존재
- SQL 대신 JPQL (Java Persistence Query Language) 사용 가능

## 2. JDBC vs JPA 비교

| 항목          | JDBC                          | JPA                          |
|--------------|------------------------------|------------------------------|
| 사용 방식     | SQL을 직접 작성               | 객체 지향적인 방식 사용       |
| 생산성       | 코드가 많아지고 복잡해짐       | 간결한 코드로 개발 속도 증가 |
| 유지보수     | SQL 변경 시 코드 수정 필요     | 변경에 유연한 구조 제공      |
| 자동화       | 수동 매핑 필요                 | 엔티티 매핑 및 자동 관리 지원 |
| 성능 최적화   | 직접 튜닝 필요                 | 1차 캐시, 지연 로딩 등 최적화 기능 제공 |
| 트랜잭션 관리 | 수동 설정                      | `@Transactional` 등 간편한 설정 |

## 3. JPA를 사용하면 좋은 점

### 3.1 생산성 증가
- SQL을 직접 작성하지 않고 객체를 사용하여 데이터를 다룰 수 있음
- 데이터베이스 변경 시 최소한의 코드 수정으로 대응 가능

### 3.2 유지보수 용이
- SQL 문법 변경 없이 비즈니스 로직을 유지할 수 있음
- 엔티티 중심의 개발이 가능하여 도메인 모델을 쉽게 이해할 수 있음

### 3.3 성능 최적화
- 1차 캐시, 지연 로딩(Lazy Loading) 등 다양한 최적화 기능 제공
- `@NamedQuery`, `Criteria API` 등을 활용하여 성능을 높일 수 있음

### 3.4 트랜잭션 및 일관성 관리
- `@Transactional` 어노테이션을 사용하여 트랜잭션을 손쉽게 관리 가능
- 영속성 컨텍스트를 통해 데이터 일관성을 유지할 수 있음

## 4. 간단한 예제 코드

### 4.1 JDBC 예제
```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

public class JdbcExample {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/testdb";
        String user = "root";
        String password = "password";
        
        try (Connection conn = DriverManager.getConnection(url, user, password);
             PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users");
             ResultSet rs = stmt.executeQuery()) {
            
            while (rs.next()) {
                System.out.println("User: " + rs.getString("username"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 4.2 JPA 예제
```java
import jakarta.persistence.*;
import java.util.List;

@Entity
class User {
    @Id @GeneratedValue
    private Long id;
    private String username;

    // Getters and Setters
}

public class JpaExample {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("example-unit");
        EntityManager em = emf.createEntityManager();

        em.getTransaction().begin();
        User user = new User();
        user.setUsername("testuser");
        em.persist(user);
        em.getTransaction().commit();

        List<User> users = em.createQuery("SELECT u FROM User u", User.class).getResultList();
        for (User u : users) {
            System.out.println("User: " + u.getUsername());
        }

        em.close();
        emf.close();
    }
}
```

## 5. 언제 JDBC를 선택해야 할까?
- 단순한 쿼리를 실행하고 별도의 ORM이 필요하지 않을 때
- 성능이 중요한 경우 직접 최적화할 수 있음
- JPA 학습이 부담스러울 때

## 6. 결론
JPA는 유지보수성과 생산성을 높이며, 다양한 자동화 기능과 최적화 기법을 제공하여 개발을 더 효율적으로 만들어준다. 하지만 단순한 애플리케이션이나 성능 최적화가 중요한 경우 JDBC가 더 적합할 수도 있다. 상황에 맞게 선택하는 것이 중요하다.

