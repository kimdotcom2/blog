# 위클리 페이퍼 4

## Spring Framework가 탄생하게 된 배경과 이를 통해 해결하고자 했던 문제점에 대해 설명하세요.

스프링 이전에 엔터프라이즈 애플리케이션 개발을 위해 EJB(Enterprise Java Beans)가 존재했다.

트랜잭션, 보안, 원격 호출 등의 비즈니스 로직과 분리된 관심사를 컨테이너에 맡기고 개발자는 핵심 로직에 집중한다는 철학을 가지고 있었다.

컨테이너는 Session Bean, Message-Driven Bean Entity Bean 등의 객체를 생성해 라이프 사이클을 관리한다.

참고 : https://docs.oracle.com/javaee/5/tutorial/doc/bnbyl.html

### EJB의 단점

1. 복잡한 설정과 높은 진입장벽

비즈니스 로직을 작성하기 위해서 필요한 많은 인터페이스와 클래스를 작성했어야 했고 XML로 설정을 관리했어야 했다.

2. 프레임워크에게 강하게 종속된 코드

EJB에서는 단순한 기능을 구현할 때도 프레임워크에서 제공하는 객체를 상속받아야만 한다.

```Java
//인터페이스 정의
import javax.ejb.Local;

@Local
public interface HelloService {
    String sayHello();
}

//EJB 클래스 구현
import javax.ejb.Stateless;

@Stateless
public class HelloServiceBean implements HelloService {
    @Override
    public String sayHello() {
        return "Hello, World!";
    }
}

//클라이언트
import javax.ejb.EJB;
import javax.servlet.http.*;
import javax.servlet.annotation.*;
import java.io.IOException;

@WebServlet("/hello")
public class HelloServlet extends HttpServlet {

    @EJB
    private HelloService helloService; // EJB 주입

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        resp.getWriter().write(helloService.sayHello());
    }
}

```

3. 객체가 프레임워크에 강하게 종속되어 다른 기술로의 이식성 저하

4. 다른 객체에 의존하지 않는 단위 테스트(Unit Test) 어려움

5. 객체가 무거워지므로 빌드 시간 저하 

6. 자바에서 추구하는 객체지향의 원칙 위배

1) 추상화된 인터페이스가 아닌 구현 클래스에 의존하므로 DIP 원칙 위배
2) EJB 클래스에 비즈니스 로직과 다른 관심사가 혼재되어 있으므로 SRP 원칙 위배
3) 컨테이너가 비즈니스 로직을 강하게 제어하므로 캡슐화 약화

### 스프링 프레임워크의 탄생

이러한 EJB의 단점을 비판하면서도 보완하기 위해 스프링의 시초가 되는 경량 프레임워크가 2002년에 공개되었다.

다른 관심사를 분리하고 비즈니스 로직에 집중한다는 EJB의 철학을 계승하면서도 보다 유연하고 가벼운 아키텍처를 지향한다.

1) POJO(Plain Old Java Object)로 클래스를 작성하고 어노테이션을 사용해 프레임워크에 종속되지 않는 객체
2) 비즈니스 로직과 다른 관심사를 분리하여 트랜잭션, 보안, 로깅 등을 처리하는 AOP(Aspect-Oriented Programming)
3) XML 대신 자바 기반 설정으로 Bean 관리
4) 개발자가 명시적으로 객체의 의존 관계를 정의하는 대신, 컨테이너가 외부에서 주입하는 IoC(Inversion of Control)


## 프레임워크와 라이브러리의 차이점을 제어 흐름의 주체와 사용 방식을 중심으로 설명하고, Spring Framework와 일반 Java 라이브러리를 예시로 들어 설명하세요.

라이브러리 (Library)

1) 개발자가 라이브러리를 호출해 소프트웨어의 흐름을 제어
2) 개발자의 코드에서 라이브러리의 API를 호출함

프레임워크 (Framework)

1) 개발자는 비즈니스 로직 등 필요한 부분만 개발하고 전체적인 흐름은 프레임워크가 제어
2) 프레임워크에서 개발자의 코드를 호출하여 프로그램을 실행함


```Java
//스프링 프레임워크의 경우

@Entity
public class Member {
    @Id
    private Long id;
    private String name;
}

public interface MemberRepository extends JpaRepository<Member, Long> {
    Member findByName(String name);
    void save(Member member);
}

@Service
public class MemberService {

    private final MemberRepository memberRepository;

    @Autowired
    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    public Member getMemberByName(String name) {
        return memberRepository.findByName(name);
    }

    @Transactional
    public void registerMember(Member member) {
        memberRepository.save(member);
    }
    
}

```

```Java
//바닐라로 개발한 경우

public class MemberRepository {

    private final String url = "jdbc:mysql://localhost:3306/mydb";
    private final String username = "root";
    private final String password = "password";

    public void registerMember(Member member) {
        Connection conn = null;

        try {
            conn = DriverManager.getConnection(url, username, password);
            conn.setAutoCommit(false); 
            
            String sql = "INSERT INTO member (id, name) VALUES (?, ?)";
            try (PreparedStatement stmt = conn.prepareStatement(sql)) {
                stmt.setLong(1, member.getId());
                stmt.setString(2, member.getName());
                stmt.executeUpdate();
            }
            
            if (member.getName().equals("error")) {
                throw new RuntimeException("강제 예외 발생");
            }
            
            conn.commit();

        } catch (Exception e) {
            
            if (conn != null) {
                try {
                    conn.rollback();
                } catch (SQLException rollbackEx) {
                    rollbackEx.printStackTrace();
                }
            }
            e.printStackTrace();
        } finally {

            if (conn != null) {
                try {
                    conn.setAutoCommit(true);
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```

