# 위클리 페이퍼 5

## 웹 서버(Web Server)와 WAS(Web Application Server)의 차이를 설명하고, Spring Boot의 내장 톰캣이 이 둘 중 어디에 해당하는지 설명해주세요.

웹 서버는 정적 컨텐츠를 서빙하는 역할을 한다.

1. 클라이언트가 서버에 Request를 전달한다.
2. 서버는 HTML, CSS, JS, 이미지 등의 정적 컨텐츠를 찾아 Response로 전달한다.

웹 애플리케이션 서버는 동적인 컨텐츠를 수행한다.

1. 클라이언트가 서버에 Request를 전달한다.
2. 서버는 Request에 따른 비즈니스 로직을 수행한다. (인증, 인가, DB 연동)
3. 서버는 비즈니스 로직의 결과를 Response로 전달한다.

Apache Tomcat = Web Server + Web Application Server (사실상 웹 애플리케이션 서버)

1. 정적 파일 전달
2. 서블릿 컨테이너에서 필터 처리
3. 디스패처 서블릿으로 핸들러 조회, 핸들러 어댑터 호출, view 선택, view 반환 

따라서 내장 톰캣은 웹 서버와 웹 애플리케이션 서버 양쪽 모두에 해당된다.


## Spring Boot에서 사용되는 다양한 Bean 등록 방법들에 대해 설명하고, 각각의 장단점을 비교하세요.

스프링은 실행 시 런타임에서 스프링 컨테이너(IoC 컨테이너)가 자동으로 Bean을 탐색하여 컨테이너에 등록하고 초기화 한다.

1. @Component 어노테이션

@Component, @Service, @Repository, @Controller 어노테이션을 사용한 클래스를 컴포넌트 스캔으로 Bean으로 등록한다.

``` Java
import org.springframework.stereotype.Service;

@Service  
public class HelloService {

    public String sayHello() {
        return "Hello, Spring Boot!";
    }
}


import com.example.demo.service.HelloService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    private final HelloService helloService;

    public HelloController(HelloService helloService) {
        this.helloService = helloService;
    }

    @GetMapping("/hello")
    public String hello() {
        return helloService.sayHello();
    }
}

```

장점
1. 매 클래스에 어노테이션을 사용하면 자동으로 스캔되기 때문에 편리하다.
2. 어노테이션의 종류로 클래스의 역할을 구분할 수 있다.

단점
1. 컨테이너가 자동으로 Bean을 등록하기 때문에 상세한 제어가 어렵다.
2. 외부 라이브러리의 클래스에는 적용할 수 없다.

2. @Configuration + @Bean 어노테이션으로 수동 Bean 등록

@Configuration 어노테이션으로 해당 클래스를 구성 클래스로 선언하고, 내부에 Bean 메소드를 정의한다.
(내부에 @Component 어노테이션이 있으므로 컴포넌트 스캔 가능)

``` Java
import com.example.demo.service.CustomService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public CustomService customService() {
        return new CustomService();
    }
}

import com.example.demo.service.CustomService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    private final CustomService customService;

    public HelloController(CustomService customService) {
        this.customService = customService;
    }

    @GetMapping("/api/hello")
    public String hello() {
        return customService.hello();
    }
}

```

장점
1. 조건부 등록, 프로파일 지정, 덮어쓰기 등 상세한 제어가 가능하다.
2. 외부 클래스를 Bean으로 등록할 수 있다.
3. 명시적으로 Bean을 지정하여 가독성이 향상된다.

단점
1. @Component 어노테이션을 사용할 때보다 코드 길이가 늘어난다.
2. 의존성 설정이 복잡해지면 클래스가 비대해진다.