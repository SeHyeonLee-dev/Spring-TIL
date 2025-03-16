# **JPA (Java Persistence API) 정리**

## **1. JPA란?**
JPA(Java Persistence API)는 자바 애플리케이션에서 관계형 데이터베이스(RDBMS)와 상호작용하는 방법을 정의한 표준 ORM (Object-Relational Mapping) 기술이다.

- **ORM(Object-Relational Mapping):** 객체와 관계형 데이터베이스를 매핑하는 기술
- **JPA는 인터페이스 기반의 표준 명세이며, 구현체가 필요함** (예: Hibernate, EclipseLink, DataNucleus 등)

## **2. JPA의 핵심 개념**
### **(1) Entity**
- 데이터베이스의 테이블과 매핑되는 클래스
- `@Entity` 어노테이션을 사용
- 기본 키는 `@Id`를 사용하여 설정

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;
}
```

### **(2) EntityManager**
- 엔티티를 데이터베이스에 저장, 수정, 삭제, 조회하는 인터페이스
- `EntityManagerFactory`를 통해 `EntityManager` 인스턴스를 생성

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("my-persistence-unit");
EntityManager em = emf.createEntityManager();
```

### **(3) 영속성 컨텍스트 (Persistence Context)**
- `EntityManager`가 관리하는 엔티티 객체의 집합
- 엔티티의 상태를 관리하며, 동일한 트랜잭션 내에서는 같은 엔티티 객체를 반환

## **3. JPA의 주요 기능**
### **(1) CRUD (Create, Read, Update, Delete)**
#### **✔️ 저장 (Create)**
```java
User user = new User();
user.setName("John Doe");
user.setEmail("john@example.com");

em.getTransaction().begin();
em.persist(user);
em.getTransaction().commit();
```

#### **✔️ 조회 (Read)**
```java
User user = em.find(User.class, 1L);
```

#### **✔️ 수정 (Update)**
```java
em.getTransaction().begin();
user.setName("Updated Name");
em.getTransaction().commit();
```

#### **✔️ 삭제 (Delete)**
```java
em.getTransaction().begin();
em.remove(user);
em.getTransaction().commit();
```

### **(2) JPQL (Java Persistence Query Language)**
JPA에서 제공하는 객체지향 SQL

```java
List<User> users = em.createQuery("SELECT u FROM User u", User.class).getResultList();
```

### **(3) 연관관계 매핑 (Association Mapping)**
#### **✔️ 일대다 (OneToMany)**
```java
@Entity
public class Department {
    @Id @GeneratedValue
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "department")
    private List<Employee> employees = new ArrayList<>();
}
```

#### **✔️ 다대일 (ManyToOne)**
```java
@Entity
public class Employee {
    @Id @GeneratedValue
    private Long id;
    private String name;
    
    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;
}
```

## **4. JPA의 장점과 단점**
### **✅ 장점**
1. **생산성 향상**: SQL 대신 객체 중심 개발 가능
2. **유지보수 용이**: 비즈니스 로직에 집중 가능
3. **성능 최적화 가능**: 1차 캐시, 지연 로딩(Lazy Loading), 배치 처리 등 지원
4. **데이터베이스 종속성 낮음**: DB 변경 시 코드 수정 최소화

### **❌ 단점**
1. **러닝 커브 있음**: 초기에 개념을 익히는 데 시간이 걸림
2. **복잡한 쿼리는 JPQL 또는 네이티브 SQL을 사용해야 함**
3. **대량 데이터 처리 시 퍼포먼스 튜닝 필요**

## **5. JPA vs MyBatis 비교**
| 비교 항목 | JPA | MyBatis |
|----------|-----|---------|
| SQL 작성 방식 | 자동 생성 (JPQL) | 직접 SQL 작성 |
| 개발 생산성 | 높음 (자동화) | SQL 작성 필요 |
| 성능 최적화 | 캐싱, 지연 로딩 등 | SQL 튜닝이 필요 |
| 유연성 | 객체 지향적 | SQL 기반으로 자유로운 튜닝 가능 |

## **6. 스프링 데이터 JPA**
### **(1) 스프링 데이터 JPA란?**
- JPA를 더 쉽게 사용할 수 있도록 도와주는 Spring Boot 기반 모듈
- `JpaRepository` 인터페이스를 사용하여 기본 CRUD 기능 제공

### **(2) 사용 예제**
#### **✔️ Repository 인터페이스 작성**
```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByName(String name);
}
```

#### **✔️ 서비스 계층에서 사용**
```java
@Autowired
private UserRepository userRepository;

public void saveUser() {
    User user = new User();
    user.setName("Alice");
    userRepository.save(user);
}
```

---

## **🔹 정리**
✔ JPA는 Java ORM 표준으로, 관계형 데이터베이스와 객체를 매핑하는 기술이다.  
✔ `EntityManager`를 사용하여 데이터를 저장, 수정, 삭제할 수 있다.  
✔ JPQL을 활용하여 객체지향 쿼리를 작성할 수 있다.  
✔ 스프링 데이터 JPA를 활용하면 `JpaRepository`를 통해 더 쉽게 JPA를 사용할 수 있다.  
✔ JPA는 SQL을 자동으로 생성하지만, 성능 최적화를 위해 JPQL 및 네이티브 쿼리를 적절히 활용해야 한다.  

---
### 📌 **추가 학습 추천**
- JPA 영속성 컨텍스트 (1차 캐시, 변경 감지, 지연 로딩)
- JPA의 다양한 연관 관계 매핑 (@OneToOne, @OneToMany, @ManyToMany)
- 스프링 데이터 JPA 활용법
- QueryDSL을 활용한 동적 쿼리 작성
