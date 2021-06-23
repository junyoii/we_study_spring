# AOP (Like C 포인터)

**1. AOP가 필요한 상황**

**2. AOP 적용**  

<br>
<br>

## 1. AOP가 필요한 상황 
1.  모든 메소드의 호출시간을 측정하고 싶다면?
2. 공통 관심 사항(cross-cutting concern) vs 핵심 관심 사항(core concern)
3. 회원 가입 시간, 회원 조회 시간을 측정하고 싶다면?

> 다음과 같은 상황을 상상해보자

![asdfasdf](https://user-images.githubusercontent.com/63052097/122994268-3d097900-d3e3-11eb-90e9-c1be74532ea9.PNG)
- 모든 메소드의 호출 시간을 측정하고 싶다면? 
= 시간 측정 로직을 각각 메소드의 시작과 끝을 측정해서 만들자
= 메소드가 1000개가 되는 상황이라도 시간 측정 로직을 만들었음 
= 초 단위에서 밀리 초로 바꿔야 한다면? 1000개를 일일이 수정해야 하는가..?

>  MemberSevice 회원 조회 시간 측정 로직 추가

```jsx
package hello.hellopspring.service;

@Transactional

public class MemberService {
    /**
       * 회원가입 메소드 
       */
    public Long join(Member member) {
          
            long start = System.currentTimeMillis(); // 현재 밀리 초를 start에 할당 

            try {
                  validateDuplicateMember(member); //중복 회원 검증
                  memberRepository.save(member);
                  return member.getId();

             } finally {   // 로직이 끝날 때 반드시 시간이 찍혀야 함(예외가 발생해도 반드시 실 
                                 행하기 위해서)  
                  long finish = System.currentTimeMillis(); // 현재 밀리 초를 finish에 할당
                  long timeMs = finish - start; // 걸린시간 
                  System.out.println("join " + timeMs + "ms"); // 반환 
             }
      }
 
      /**
        * 전체 회원 조회 메소드 
        */
       public List<Member> findMembers() { 

              long start = System.currentTimeMillis(); // 시작 밀리 초 측정 

              try {
                  return memberRepository.findAll();
               } finally {
                    long finish = System.currentTimeMillis(); // 마지막 밀리 초 측정 
                    long timeMs = finish - start; //걸린 시간 
                    System.out.println("findMembers " + timeMs + "ms"); // 반환  
                }
         }
}
```
> 결과

![회원 가입](https://user-images.githubusercontent.com/63052097/123037115-12421380-d429-11eb-9002-df2d51c25b06.PNG)

회원 가입을 실행하면 

![회원가입 결과](https://user-images.githubusercontent.com/63052097/123037116-13734080-d429-11eb-839e-09cf8d4ec3ef.PNG)

join = 25ms

![회원목록 클릭](https://user-images.githubusercontent.com/63052097/123037117-13734080-d429-11eb-9183-5c354f8ff8c0.PNG)

회원 목록 링크를 클릭해서 전체 회원 조회 메소드 실행하면 

![전체 회원 조회](https://user-images.githubusercontent.com/63052097/123037133-19692180-d429-11eb-974e-f60616dbd280.PNG)

findmemebers =1ms

 ※잠깐 
- 메소드를 처음 돌리면 메소드랑 클래스 메타 데이터 이것저것이 로딩되어서 오래걸린다고 합니다! 두 번째 실행하면 시간이 단축됨!  
- 실제 운영에서 성능을 많이 받아야 하는 서버는 미리 메소드를 이것저것 호출하는 'warm up' 과정을 한다고 합니다!  

> 문제점 

- 회원가입, 회원 조회에 시간을 측정하는 기능은 **핵심 관심 사항**이 아니다 
= 각 메소드의 핵심 관심 사항은 각 try 내부에 있는 로직이지!(즉 회원가입 메소드는 회원가입시키는 로직!! ) finally에 있는 시간 측정은 부가적인 로직임.. 
- 시간을 측정하는 로직은 **공통 관심 사항**이다. 
= 공통적으로 여러 메소드에서 사용하는 공통 기능이다! 
- 시간을 측정하는 로직과 핵심 비즈니스의 로직이 섞여서 유지보수가 어렵다.
= 위의 코드를 보시라.. 유지보수가 겁나게 힘들다.. 
- 시간을 측정하는 로직을 별도의 공통 로직으로 만들기 매우 어렵다.
= 물론 finally 부분의 시간 간격을 계산하고 출력하는 로직은 별도의 로직으로 만드는 것은 가능.. 그러나 시작 시간 측정 부터 finally까지 모든 과정을 별도의 로직을 만드는 것은 어려운 일 (메소드마다 언제 끝날지 알 수 없으니!!)
- 시간을 측정하는 로직을 변경할 때(예를 들어 ms ->s라던지 등등) 모든 로직을 찾아가면서 번경해야 한다 
= 이런 식으로 999개를 일일이 만들면 됩니다^^ 

## 2. AOP 적용
> AOP란?

**Aspect Oriented Programming** 
= 관점 지향 프로그래밍 
= 공통 관심 사항(cross-cutting concern) vs 핵심 관심 사항(core concern) 분리  
= 반복 사용되는 로직들을 모듈화 하여 필요할때 호출해서 사용하는 방법 (https://velog.io/@gwontaeyong/Spring-AOP%EC%97%90%EC%84%9C-Proxy%EB%9E%80)

![스프링 컨테이너](https://user-images.githubusercontent.com/63052097/123046729-6a801200-d437-11eb-8db4-c0d6ce369be1.PNG)

이전에는 시간 측정 로직을 각 메소드에 적용했는데 시간 측정 로직(공통 관심 사항)을 한 군데에 모으고 내가 원하는 메소드에 지정하면 된다!!



> AOP 적용 

![image](https://user-images.githubusercontent.com/63052097/123079828-416f7980-d457-11eb-9e29-80db28dc83b7.png)

 aop패키지를 만들고 TimeTraceAop  클래스를 생성! 

```jsx 

package hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint; // ProceedingJoinPoint를 가져오면서 import
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect; // @Acpect 어노테이션으로 인해 import
import org.springframework.stereotype.Component;

@Component 
// 본래는 TimeTraceAop를 Bean으로 등록하는 걸 더 선호하는 편? = 더 특별해보임!   
//  AOP라는 걸 인지할 수 있도록 Bean에 등록해주면 좋다
// 여기선 걍 Component 쓰겠다^^ 
(https://atoz-develop.tistory.com/entry/Spring-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B9%88Bean%EC%9D%98-%EA%B0%9C%EB%85%90%EA%B3%BC-%EC%83%9D%EC%84%B1-%EC%9B%90%EB%A6%AC)

@Aspect // 이 Aspect 어노테이션이 있어야 AOP이 가능! 
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")
    // 공통관심사항을 타겟팅 해줘야 한다
    // 실행(리턴타입 - 패기지명 - 하위에 있는 모든 파일.디렉 등 - 클래스 이름(매개변수 타입))
    // 즉 hello.hellospring 하위에 있는 모든 클래스 메소드에 다 적용해라 
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {

                 long start = System.currentTimeMillis(); // 시작 밀리 초 (이전과 동일)

                 System.out.println("START: " + joinPoint.toString()); 
                // 어떤 메소드를 call했는지 알아보도록 
  
                 
                 try {
                               return joinPoint.proceed(); 
                              // 다음 메소드로 진행하도록 받는 것
                             // Object result = joinPoint.proceed(); return result;를 인라인 코드로 작 
                                 성한 것임  
                  } finally {
                         long finish = System.currentTimeMillis(); // 끝 밀리 초(이전과 동일)
                         long timeMs = finish - start; // 시간 측정(이전과 동일) 

                         System.out.println("END: " + joinPoint.toString()+ " " + timeMs + "ms");
                }
        }
}
```

**요약**

**execution(~) 에서 메소드 호출이 될 때마다 execution(ProceedingJoinPoint joinPoint)의 JoinPoint에서 가로챈다(인터셉팅) **

**참고**
- (https://engkimbs.tistory.com/746) = 참고하세요! AOP 적용에 대해 참고한 부분!! 
- JointPoint : Advice(공통 관심 사항 구현부분)가 적용될 위치, 끼어들 수 있는 지점. 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용가능

출처: https://engkimbs.tistory.com/746 [새로비]

> 결과 
 
![image](https://user-images.githubusercontent.com/63052097/123050045-3575be80-d43b-11eb-8345-e8c33ba1e974.png)

회원 목록 클릭 후 확인해보면

![image](https://user-images.githubusercontent.com/63052097/123050506-c056b900-d43b-11eb-93f6-070a25785fc7.png)

Controller, Service, repository가 실행되는 게 들어가고 끝날 때 시간이 출력 

> 해결

-  회원가입, 회원 조회등 핵심 관심사항과 시간을 측정하는 공통 관심 사항을 분리!
-  시간을 측정하는 로직을 별도의 공통 로직으로!
-  **핵심 관심 사항**을 깔끔하게 유지할 수 있다.
-  변경이 필요하면 이 로직만 변경하면 된다!
-  원하는 적용 대상을 선택할 수 있다. 
= execution(~)에서 원하는 조건을 여러개 조합해서 적용할 수 있다! 보통 package 레벨에서 많이 작성함 
= 만약 * hello.hellospring.service..*(..))라고 작성한다면 service하위에 있는 애덜만 작동한다! 위와 똑같은 동작을 해도 다음과 같이 service 하위에 있는 메소드의 결과가 출력된다. 

![image](https://user-images.githubusercontent.com/63052097/123072067-2f3e0d00-d450-11eb-8a82-196734398fd7.png)

> 스프링의 AOP 동작 방식 설명 


**1. AOP 적용 전 의존관계** 

![image](https://user-images.githubusercontent.com/63052097/123072567-9fe52980-d450-11eb-893d-4fa052f5898e.png)

- Controller와 memberService를 의존
- Controller에서 메소드를 호출하면 memberService에서도 join을 호출할 것임

**2. AOP 적용 후 의존관계** 

![image](https://user-images.githubusercontent.com/63052097/123072747-c30fd900-d450-11eb-9492-63c170358631.png)

- AOP를 적용하고 execution(* hello.hellospring..*(..))에서 어디에 적용할지 지정하면 특정 서비스가 지정되는데 
- **이 때 스프링은 AOP가 적용되어 있으면 가짜 memberService 즉 '프록시'를 만든다.**
- 스프링 컨테이너는 다음과 같이 동작. 
1. 컨테이너에서 Spring Bean을 등록할 때 프록시(가짜 spring bean)을 앞으로 세워둔다 
2. 컨트롤러는 프록시 memberService를 주입 -> 컨트롤러가 프록시 memberService를 호출 -> 프록시 memberService가 실제 memberService를 호출-> 프록시가 끝나면 joinPoint.proceed()가 호출될 때 진짜 memberService를 호출 
- 프록시 기술에 대한 더 자세한건 스프링 핵심강의에서... 

> 프록시가 주입되는지 확인해보기 

![image](https://user-images.githubusercontent.com/63052097/123078101-9b6f3f80-d455-11eb-8e2f-974e4e3d34cb.png)

결과

![image](https://user-images.githubusercontent.com/63052097/123078197-b2159680-d455-11eb-887b-425ed6db4e9e.png)

**MemberService에서 끝나는게 아니라 MemberService$$EnhancerBySpringCGLIB$$de89008 이라고** 출력됨
MemberService를 가지고 복제해서 코드를 조작한 기술 = 프록시 임을 알 수 있다!   

**3. AOP 적용 전 전체그림**

![image](https://user-images.githubusercontent.com/63052097/123072779-ca36e700-d450-11eb-9e9f-2735a72e3bf4.png)

**4. AOP 적용 전 전체그림**

![image](https://user-images.githubusercontent.com/63052097/123072847-dc188a00-d450-11eb-8e63-67de646756ac.png)


**추가공부 및 정리**
- AOP = proxy라는 가상의 bean 인스턴스(?)로서 작동되어지는 것 
- Bean (https://velog.io/@gwontaeyong/Spring-AOP%EC%97%90%EC%84%9C-Proxy%EB%9E%80)
= new 연산자로 어떤 객체를 생성했을 때가 아닌 ApplicationContext.getBean()으로 얻어질 수 있는 객체를 Bean이라고 한다.
한마디로 ApplicaitonContext가 알고 있는 객체, 즉 ApplicationContext가 만들어서 그 안에 담고 있는 객체를 의미 

 
