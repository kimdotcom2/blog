# 위클리 페이퍼 7

## 웹 API의 발전 과정에서 SOAP에서 REST로의 전환이 일어난 이유와 그 장단점에 대해 설명하세요.

### SOAP(Simple Object Access Protocol)란?

다른 언어로 빌드된 애플리케이션끼리 통신할 수 있도록 하는 웹 서비스 프로토콜

기존의 XML-RPC에서 발전된 패턴

HTTP, SMTP 등을 통해 XML 기반의 메시지를 주고 받음

```SOAP
<?xml version="1.0"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope" 
                     soap:encodingStyle=" http://schemas.xmlsoap.org/soap/encoding">
  <soap:Header>
                ...
  </soap:Header>
  <soap:Body>
               ...
    <soap:Fault>
                 ...
    </soap:Fault>
  </soap:Body>
</soap:Envelope>

```

장점
1) 당시 기준으로 높은 이식성
2) WS-Security라는 토큰을 이용한 메시지 레벨 보안
3) 트랜잭션 지원
4) HTTP 외에도 다양한 프로토콜 지원 (SMTP, FTP...)

단점
1) 복잡한 XML 구조
2) 큰 데이터로 발생하는 오버헤드
3) 브라우저에서 캐싱 불가능
4) 높은 진입장벽

## REST((Representational State Transfer)로의 전환

보다 가볍게 설계된 API의 필요성이 떠오름

대안으로 제시된 REST는 HTTP 프로토콜의 단순성과 유연성을 활용한 API

1) XML 외에도 더 가볍고 유연한 JSON 지원
2) HTTP method(GET, POST, PUT, DELETE...)를 활용한 제약 조건
3) 표준화 대신 유연함
4) SOAP에 비해 가볍고 경량화된 기능


## Spring Boot에서 @RestController로 들어온 HTTP 요청이 처리되어 응답으로 변환되는 전체 과정을 설명하세요. 특히 HTTP 메시지 컨버터가 동작하는 시점과 역할을 포함해서 설명하세요.

1) 클라이언트가 서버에 Request

```JSON
Content-Type: application/json
{
  "name": "Bob",
  "age": 25,
  "email" : "bob@mail.com"
}
```

2) 서블릿 필터 체인
3) 디스패처 서블릿이 전달받아 핸들러(컨트롤러) 탐색
4) 디스패처 서블릿이 핸들러 어댑터에 전달
5) 핸들러 어댑터가 RestController의 메소드에 요청 위림
6) ArgumentResolver 컴포넌트가 메소드의 인자에 맞추어 컨트롤러의 파라미터를 채움
7) HttpMessageConverter가 요청을 역직렬화 하여 객체로 변환
```Java

public class Person {
  private String name;
  private int age;
  private String email;
  
  public Person() {}
  
  public Person(String name, int age, String email) {
    this.name = name;
    this.age = age;
    this.email = email;
  }
  
  public String getName() {
    return name;
  }
  public void setName(String name) {
    this.name = name;
  }

  public int getAge() {
    return age;
  }
  public void setAge(int age) {
    this.age = age;
  }

  public String getEmail() {
    return email;
  }
  public void setEmail(String email) {
    this.email = email;
  }
  
  @Override
  public String toString() {
    return "Person{name='" + name + "', age=" + age + ", email='" + email + "'}";
  }
}

Person{name='Bob', age=25, email='bob@mail.com'}
```
8) 비즈니스 로직 처리
9) HttpMessageConverter가 객체를 json으로 직렬화
10) 컨트롤러가 Response 반환(컨트롤러 -> 인터셉터 -> 디스패처 서블릿 -> 서블릿 필터)
