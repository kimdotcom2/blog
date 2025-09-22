# 위클리 페이퍼 6

##  Spring에서 AOP(Aspect Oriented Programming)가 필요한 이유와 이를 활용한 실제 애플리케이션 개발 사례에 대해 설명하세요.

### AOP(Aspect Oriented Programming)란?

AOP란 여러 모듈에서 반복되는 공통적인 관심사를 하나의 관점(Aspect)으로 보고,
이를 비즈니스 로직에 분리하여 하나의 모듈로 만드는 프로그래밍 패러다임이다.

대표적인 관심사
- 로그
- 트랜잭션
- 보안(인증, 인가)
- 예외 핸들링

스프링에서는 Spring AOP를 사용해 쉽게 Bean에 AOP를 구현할 수 있다.
(더 복잡한 로직의 경우 AspectJ 라이브러리 사용)

### 코드 비교

- 비즈니스 로직과 공통 로직이 섞여 있는 코드

```Java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;

public class MemberServiceManualTx {

    public void registerMember(String name) {
        Connection conn = null;
        try {
            conn = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/testdb", "root", "password");
            conn.setAutoCommit(false); // 🔸 트랜잭션 시작

            // 회원 저장
            String sql = "INSERT INTO member (name) VALUES (?)";
            try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
                pstmt.setString(1, name);
                pstmt.executeUpdate();
            }

            // 로그 저장
            String logSql = "INSERT INTO log (message) VALUES (?)";
            try (PreparedStatement pstmt = conn.prepareStatement(logSql)) {
                pstmt.setString(1, "회원 등록: " + name);
                pstmt.executeUpdate();
            }

            conn.commit(); // 🔹 트랜잭션 커밋
        } catch (Exception e) {
            if (conn != null) {
                try {
                    conn.rollback(); // 🔻 롤백
                } catch (Exception rollbackEx) {
                    rollbackEx.printStackTrace();
                }
            }
            e.printStackTrace();
        } finally {
            if (conn != null) {
                try {
                    conn.close(); // 🔚 연결 해제
                } catch (Exception closeEx) {
                    closeEx.printStackTrace();
                }
            }
        }
    }
}
```

- 트랜잭션 처리를 Aspect로 분리한 코드

```Java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class MemberServiceSpringTx {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Transactional // 🔸 트랜잭션 시작
    public void registerMember(String name) {
        // 회원 저장
        jdbcTemplate.update("INSERT INTO member (name) VALUES (?)", name);

        // 로그 저장
        jdbcTemplate.update("INSERT INTO log (message) VALUES (?)", "회원 등록: " + name);
        // 🔹 예외가 발생하면 자동으로 롤백됨
    }
}
```

장점
1) 가독성 향상
2) 관심사의 분리로 SRP 준수
3) 유지보수성 향상
4) 재사용 용이

##  Spring MVC에서 클라이언트의 요청 처리 흐름을 @Controller와 @RestController의 차이점을 중심으로 각각의 처리 과정과 특징을 포함하여 설명하세오.

###  스프링 MVC에서의 요청 처리 흐름

1. 클라이언트가 서버에 Request를 보냄
2. WAS의 서블릿 컨테이너의 필터 체인을 거침(필요 시 스프링 컨테이너의 필터에 위임)
3. 디스패처 서블릿(프론트 컨트롤러)가 스프링 컨테이너의 핸들러를 조회
4. 핸들러는 알맞은 메소드를 디스패처 서블릿에 전달
5. 디스패처 서블릿은 알맞은 핸들러 어댑터를 호출
6. 어댑터가 컨트롤러를 호출
7. 웹 애플리케이션에서 비즈니스 로직 수행 
8. 컨트롤러가 알맞은 Response를 반환

### @Controller vs @RestController

@Controller
1. 논리적 view를 실제 파일과 매핑하여 view 반환
2. ViewResolver 사용
3. 웹 페이지 렌더링

@RestController
1. 객체를 JSON으로 반환
2. MessageConverter
3. REST API 개발

