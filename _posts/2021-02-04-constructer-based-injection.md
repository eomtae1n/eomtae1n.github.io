---
title: 필드 주입보다 생성자 주입을 권장하는 세 가지 이유
layout: post
date: 2021-02-04
description: A complete post.
image: 
categories: ["n"]
---


안녕하세요!

스프링 교육시간에 언급해주신 DI 방법 중 `@Autowired` 어노테이션만을 이용한 필드 주입과 생성자 주입 방법의 차이점을 찾아 정리해보았습니다.
결론부터 말씀드리면, 스프링에서는 필드 주입(Field Injection), 수정자 주입(Setter based Injection) 방법보다 생성자 주입(Constructor based Injection) 방법을 권장하고 있습니다.
(이 글에서는 두 방법의 명확한 차이를 보이기 위해 수정자 주입 방법의 설명은 생략했습니다. <b>*참고문서*</b>를 확인해주세요.)

## **필드 주입보다 생성자 주입을 권장하는 세 가지 이유**

### **1\. 주입하려는 필드를 final 키워드로 선언해 오류를 방지할 수 있다**
<br>
* **필드 주입**

``` java
@Controller
public class GuestbookController {

  @Autowired
  private GuestbookService service;

  public void someMethod() {
    service = null;       // 이런 경우는 드물겠지만, final이 아니기에 값을 변경할 수 있습니다.
    service.register();   // NullPointerException 오류!
  }

}
```

* **생성자 주입**

``` java
@Controller
public class GuestbookController {

  private final GuestbookService service;

  @Autowired
  public GuestbookController(GuestbookService service) {
    this.service = service;
  }

  public void someMethod() {
    service = null;       // 컴파일 시점에 에러 발생!
  }

}
```

필드 주입의 경우에는 `final`로 선언이 불가합니다. 따라서 런타임 도중 객체를 변경할 경우`NullPointerException`과 같은 오류가 발생할 수 있습니다.
하지만 생성자 주입의 경우, 해당 필드를 `final`로 선언해 오류를 컴파일 시점에 방지할 수 있습니다.

### **2\. 순환 참조를 방지할 수 있다**
<br>
개발 시 여러 객체간의 의존관계가 복잡해지면, A가 B를 참조하고, 다시 B가 A를 참조하는 순환참조가 발생할 수 있습니다.

* **필드 주입**

``` java
@Service
public class CourseServiceImpl implements CourseService {

    @Autowired
    private StudentService studentService;

    @Override
    public void courseMethod() {
        studentService.studentMethod();   // studentMethod() 호출
    }
}
```

``` java
@Service
public class StudentServiceImpl implements StudentService {

    @Autowired
    private CourseService courseService;

    @Override
    public void studentMethod() {
        courseService.courseMethod();   // 다시 courseMethod() 호출
    }
}
```
<br>
`CourseServiceImpl`의 `courseMethod()`는 `StudentServiceImpl`의 `studentMethod()`를 호출하고,
`StudentServiceImpl`의 `studentMethod()`는 `CourseServiceImpld`의 `courseMethod()`를 호출하고 있습니다.
문제는 실제로 코드가 호출되기 전까지 오류없이 돌아가기때문에, 순환참조 오류를 방지할 수 없다는 것입니다. 이 경우는 서로 호출을 반복하다 StackOverflowError를 발생시킵니다.

* **생성자 주입**

``` java
@Service
public class CourseServiceImpl implements CourseService {

    private final StudentService studentService;

    @Autowired
    public CourseServiceImpl(StudentService studentService) {
        this.studentService = studentService;
    }

    @Override
    public void courseMethod() {
        studentService.studentMethod();
    }
}
```

``` java
@Service
public class StudentServiceImpl implements StudentService {

    private final CourseService courseService;

    @Autowired
    public StudentServiceImpl(CourseService courseService) {
        this.courseService = courseService;
    }

    @Override
    public void studentMethod() {
        courseService.courseMethod();
    }
}
```

이 경우는 위의 코드에서 필드 주입을 생성자 주입으로만 바꾼 코드입니다. 실행해보면 아래와 같은 로그가 찍히며 앱 구동이 실패합니다.
<img width="367" alt="순환참조 오류" src="https://user-images.githubusercontent.com/37218734/107149498-48ac5400-699c-11eb-805a-2e06148fd71b.png">
스프링 앱 구동 시 빈을 생성하는 시점에서 객체 간 사이클관계가 형성되어 위와 같이 안내를 해주기에 앱 실행도중 종료되는 것을 방지합니다.

### **3\. 테스트 코드 작성이 용이하다**
<br>
테스트 코드 작성에 대한 연습은 앞으로 더 필요하기때문에 세 번째 장점은 아직 잘 체감되지 않는 것 같습니다.

하지만,
<img width="377" alt="테스트코드 작성 용이" src="https://user-images.githubusercontent.com/37218734/107149537-72657b00-699c-11eb-9e6f-bd3ad4fb7194.png">
이런 장점이 있다고 합니다. 이 부분은 이후에 테스트 코드를 작성해보며 조금 더 이해해보겠습니다.

## **결론**

스프링에서 `@Autowired`만을 이용해 객체에 의존성을 주입하는 필드 주입 방법보다 생성자를 이용한 주입을 권장하는 이유는,

* `**final**`**키워드를 이용해 런타임 환경에서 객체 변경 방지**
* **앱 실행도중 순환참조로 인한 강제종료 방지**
* **테스트 코드 작성 용이**

입니다.
혹시 틀린부분이나 추가사항이 있다면 댓글 부탁드립니다. 이상입니다!

## 참고문서

[https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)
[https://madplay.github.io/post/why-constructor-injection-is-better-than-field-injection](https://madplay.github.io/post/why-constructor-injection-is-better-than-field-injection)