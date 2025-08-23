# 스프린트 2

## 객체지향 프로그래밍에서 '단일 책임 원칙(SRP)'과 '개방-폐쇄 원칙(OCP)'에 대해 설명하고, 각각의 원칙을 적용한 코드 예시를 들어주세요.

### SRP(Single Responsibility Principle)

하나의 클래스(또는 모듈, 객체)는 하나의 책임만을 가진다는 원칙을 의미한다.

여기서 책임이란 모듈의 기능을 담당한다.

만약 클래스가 변경되어야 한다면 그 이유는 반드시 한가지 기능(책임) 때문이어야 한다.

클래스가 여러 역할을 담당하고 있거나 책임이 모호하다면 구조가 복잡해져 유지보수에 문제가 생기기 때문이다.


SRP를 준수하지 않은 코드

```Java
public class Employee {
private String name;
private int salary;

    public Employee(String name, int salary) {
        this.name = name;
        this.salary = salary;
    }

    // 직원 정보 출력
    public void printEmployeeInfo() {
        System.out.println("Name: " + name);
        System.out.println("Salary: " + salary);
    }

    // 급여 계산 로직
    public int calculateAnnualSalary() {
        return salary * 12;
    }

    // 직원 정보를 DB에 저장
    public void saveToDatabase() {
        System.out.println("Saving " + name + " to database...");
        // 이하 DB 저장 로직
    }
}
```


SRP를 준수하도록 리팩토링한 코드

```Java
public class Employee {
    private String name;
    private int salary;

    public Employee(String name, int salary) {
        this.name = name;
        this.salary = salary;
    }

    public String getName() {
        return name;
    }

    public int getSalary() {
        return salary;
    }
}

public class SalaryCalculator {
    public int calculateAnnualSalary(Employee employee) {
        return employee.getSalary() * 12;
    }
}

public class EmployeePrinter {
    public void printEmployeeInfo(Employee employee) {
        System.out.println("Name: " + employee.getName());
        System.out.println("Salary: " + employee.getSalary());
    }
}

public class EmployeeRepository {
    public void saveToDatabase(Employee employee) {
        System.out.println("Saving " + employee.getName() + " to database...");
        // DB 저장 로직 (가정)
    }
}
```

### OCP(Open-Closed Principle)

객체(또는 모듈, 클래스)는 확장에는 열려있으면서도 수정에는 닫혀있어야 하는 원칙을 의미한다.

만약 새로운 기능을 만들어야 할 때, 기존의 코드를 대거 수정해야 한다면 유지보수가 복잡해지게 된다.

객체지향의 핵심 원리인 추상화를 통해 기능을 정의하고, 상세한 구현은 구현체 클래스나 인터페이스에게 맡겨 확장하게 만든다면 쉽게 문제를 해결할 수 있다.


OCP를 준수하지 않은 코드

```Java
public class User {
    private int id;
    private String name;

    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }
}

public class UserRepository {
    public void save(User user) {
        // MySQL 전용 코드
        System.out.println("Saving user to MySQL: " + user.getName());
    }

    public User findById(int id) {
        // MySQL 전용 코드
        System.out.println("Finding user in MySQL with id: " + id);
        return new User(id, "MySQLUser");
    }
}
```


OCP를 준수하도록 리팩토링한 코드

```Java
public interface UserRepository {
    void save(User user);
    User findById(int id);
}

public class MySQLUserRepository implements UserRepository {
    public void save(User user) {
        System.out.println("Saving user to MySQL: " + user.getName());
    }

    public User findById(int id) {
        System.out.println("Finding user in MySQL with id: " + id);
        return new User(id, "MySQLUser");
    }
}

public class PostgreSQLUserRepository implements UserRepository {
    public void save(User user) {
        System.out.println("Saving user to PostgreSQL: " + user.getName());
    }

    public User findById(int id) {
        System.out.println("Finding user in PostgreSQL with id: " + id);
        return new User(id, "PostgreSQLUser");
    }
}

public class Main {
    public static void main(String[] args) {
        User user = new User(1, "Alice");

        // 필요에 따라 구현체를 교체
        UserRepository repo = new MySQLUserRepository();
        // UserRepository repo = new PostgreSQLUserRepository();
        // UserRepository repo = new MongoDBUserRepository();

        repo.save(user);
        User found = repo.findById(1);
        System.out.println("Found user: " + found.getName());
    }
}