# 📌 Spring에서 Bean, IoC, DI 개발 정보

---

## 1️⃣ Spring에서 Bean, IoC, DI 개발 이해

### 🔹 Bean이란?
- Spring이 관리하는 객체.
- 개발자가 직접 `new`를 호출할 필요가 없음.
- `@Component`, `@Service`, `@Repository`, `@RestController` 등을 사용하여 등록 가능.

### 🔹 IoC (Inversion of Control, 제어의 여전)
- 객체의 생성과 관리 권한을 Spring이 담당함.
- 개발자가 직접 객체를 생성하지 않고, Spring 컨테이너가 관리.
- `MemberService`가 `MemberRepository`를 직접 생성하지 않고, **Spring이 대신 주입**.

### 🔹 DI (Dependency Injection, 의종성 주입)
- 필요한 Bean을 Spring이 자동으로 주입해 줍.
- `@Autowired` 없이 **생성자 주입 방식**을 주로 사용.

```java
public class MemberController {
    private final MemberService memberService;

    public MemberController(MemberService memberService) { // 🔥 생성자 주입
        this.memberService = memberService;
    }
}
```

---

## 2️⃣ Spring Bean은 요청만부터 새로운 객체가 필요할까?

### 🔹 Bean은 기본적으로 싱글톤(Singleton)
- Spring이 실행되는 때 한 번만 객체를 생성하고 **모든 요청에서 동일한 인스턴스를 재사용**함.
- `@RestController`, `@Service`, `@Repository`가 붙은 클래스들은 기본적으로 **싱글톤**로 관리됨.

#### ✅ 싱글톤으로 관리되는 이유
1. **매 요청만부터 객체를 생성하면 메모리 낭지가 심함** → 한 번만 생성하여 효율적으로 관리.
2. **객체 간 의종성 관리가 쉽음** → 같은 인스턴스를 여러 공간에서 재사용 가능.
3. **Spring 컨테이너(Application Context)가 자동으로 관리**.

---

## 3️⃣ DTO와 Entity는 왜 Bean으로 등록하지 않을까?

### 🔹 DTO(`MemberRequestDto`)는 Bean이 아니다
- 요청만부터 다른 데이터를 담아야 하미로 **매번 새로운 객체가 필요**.
- Spring이 관리하는 싱글톨 Bean으로 만드면 데이터가 꿈일 위험 위에 있음.

```java
public class MemberRequestDto {
    private String name;
    private String email;
}
```

---

## 4️⃣ 요청만부터 새로운 객체를 만들고 싶다면?
기본적으로 Spring Bean은 싱글톨이지만, 매번 새로운 객체를 만들고 싶다면 `@Scope("prototype")`을 사용.

```java
@Service
@Scope("prototype")
public class MemberService { }
```

✅ 하지만 대부분의 경우 **싱글톨이 더 효율적**이다.

