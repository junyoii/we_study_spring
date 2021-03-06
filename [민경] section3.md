# 회원 관리 예제 - 백앤드 개발

1. 비즈니스 요구사항 정리
2. 회원 도메인과 리포지토리 만들기 
3. 회원 리포지토리 테스트 케이스 작성
4. 회원 서비스 개발 
5. 회원 서비스 테스트

## **1. 비즈니스 요구사항 정리**

- 회원ID / 이름을 등록 / 조회하는 서비스 제공
- 아직 데이터 저장소가 선정되지 않았다는 가정

→ NoSQL , RDB 등등 어떤 저장소를 할지 고민중

 (참고 : [https://velog.io/@wlsgh46/RDBMS-NOSQl-차이](https://velog.io/@wlsgh46/RDBMS-NOSQl-%EC%B0%A8%EC%9D%B4) )

### 일반적인 웹 Application 계층 구조
![spring1](https://user-images.githubusercontent.com/69148373/117969091-f47d8b00-b361-11eb-82a5-753070d6f639.png)

- 컨트롤러 : MVC 패턴에서의 컨트롤러 역할
- 서비스 : 비즈니스 도메인 객체를 가지고 핵심 비스니스 로직이 동작하게 하는 객체 (ex - 로그인, 중복 회원 로그인 금지 등)
- 리포지토리 : 데이터베이스에 접근하여 도메인 객체를 DB에 저장하고 관리
- 도메인 : 비즈니스 도메인 객체  (ex - 회원, 주문, 쿠폰 등등)
- 아래의 그림처럼,  리포지토리를 인터페이스로 구현

 : 아직 데이터 저장소가 선정되지 않았기때문에, 후에 구현 클래스를 변경할 수 있도록 설계 

![spring2](https://user-images.githubusercontent.com/69148373/117969337-3c9cad80-b362-11eb-9c96-f5ba15b5f4e8.png)

# 2. **회원 도메인과 리포지토리 만들기**

비즈니스 요구사항이 회원의 ID와 이름을 등록하고 조회하는 것. 

→ 회원의 ID와 이름을 GET/SET할 수 있는 회원객체(도메인) 만들기

→ 회원 객체를 DB에 저장하고 조회할 수 있는 리포지토리  만들기 

### 회원 도메인

회원객체

```jsx
package hello.hellospring.domain;

public class Member{ 
	private Long id;
	private String name;

	public Long getId(){
		return id;
	}	
	public void setId(Long id){
		this.id = id;
	}
	
	public String getName(){
		return name;
	}
	public void setName(String name){
		this.name = name;
	}
}
```

- ID는 사용자가 만드는 ID가 아니라 시스템적으로 구분하기 위한 ID

### **회원 리포지토리 만들기**

1. 회원 리포지토리 인터페이스 

```jsx
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
import java.util.List;
import java.util.Optional;

public interface MemberRepository {
		Member save(Member member); // 회원 DB에 저장하기 
    Optional<Member> findById(Long id); // DB에서 회원 ID 조회
    Optional<Member> findByName(String name); // DB에서 회원 NAME 조회 
    List<Member> findAll(); // DB에 저장된 모든 회원 조회 
} 
```

- Optional은 "null이 될 수도 있는 객체"를 감싸고 있는 class
- Optional로 객체를 감싸서 사용하면 아래의 장점 가짐.
    1. NullPointerException(NPE) 예외를 유발할 수 있는 null을 직접 다루지 않아도 된다. 
    2. null 체크를 직접 하지 않아도 된다. 
    3. 명시적으로 해당 변수가 null일 수도 있다는 가능성을 표현할 수 있어 불필요한 방어 logic을 줄일 수 있다.

(참고 : [https://www.daleseo.com/java8-optional-after/](https://www.daleseo.com/java8-optional-after/))

2. 회원 리포지토리 메모리 구현체 

- 단축키 : alt + enter - method 자동으로 implement 가능

```jsx
package hello.hellospring.repository;
import hello.hellospring.domain.Member;
import java.util.*;

public class MemoryMemberRepository implements MemberRepository {
	private static Map<Long, Member> store = new HashMap<>();
	private static long sequence = 0L;

  @Override
  public Member save(Member member) {
 	 member.setId(++sequence); //회원 수 증가할수록 id 하나씩 올려주기 
	 store.put(member.getId(), member);
	 return member;
  }

  @Override
  public Optional<Member> findById(Long id) {
		return Optional.ofNullable(store.get(id)); //store에서 해당 id 꺼내기(get)
    //해당 id가 없을수도 있기 때문에 optional로 처리 
	}

  @Override
  public List<Member> findAll() {
	  return new ArrayList<>(store.values()); //store에 있는 member값들을 list로 반환 
  }

  @Override
  public Optional<Member> findByName(String name) {
  	 return store.values().stream()
  	 .filter(member -> member.getName().equals(name)) //DB에 저장된 회원 이름이 Parameter로 넘어온 이름과 같은지 확인 
  	 .findAny(); // 같은 값만 필터링되어 findAny() -> 결과가 하나라도 있으면 반환. 없으면 optional이 null 값 처리 
  }

  public void clearStore() {
 	 store.clear(); // test할때 사용
  }
}
```

## 3. **회원 리포지토리 테스트 작성**

작성한 코드가 정상적으로 동작하는지를 테스트 하기 위해  test case 작성 

- Test 방법
1. 자바의 main method를 통해 실행 
2. 웹 애플리케이션의 컨트롤러 통해 해당 기능 실행 

    → 실행 오래걸리고, 반복 실행 어렵고, 한번에 여러 테스트 실행하기 어렵다는 단점

 3. JUnit이라는 프레임워크로 테스트 실행 

    → TEST code를 작성해서 지금까지 작성한 기능 code를 검증해보는 것

### 회원 리포지토리 메모리 구현체 테스트

- 단축키 : shift + F6 - Rename

```jsx
package hello.hellospring.reporsitory;

import hello.hellospring.domain.Member;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;
import java.util.List;
import java.util.Optional;
import static org.assertj.core.api.Assertions.*;

class MemoryMemberRepositoryTest {
 MemoryMemberRepository repository = new MemoryMemberRepository();
 
 @AfterEach
 public void afterEach() {
	 repository.clearStore();
 }

 @Test
 public void save(){
	//given 
  Member member = new Member();
  member.setName("spring");
  
  //when 
  repository.save(member);

  //then 
  Member result = repository.findById(member.getId()).get();
  asserThat(member).isEqualTo(result);
}

@Test
public void findByName() {
  //given
  Member member1 = new Member();
  member1.setName("spring1");
  repository.save(member1);
  
  Member member2 = new Member();
  member2.setName("spri2");
  repository.save(member2);
   
  //when
  Member result = repository.findByName("spring1").get();
 
  //then
  assertThat(result).isEqualTo(member1);
 }

 @Test
 public void findAll() {
  //given
  Member member1 = new Member();
  member1.setName("spring1");
  repository.save(member1);

  Member member2 = new Member();
  member2.setName("spring2");
  repository.save(member2);

  //when
  List<Member> result = repository.findAll(); 
  
  //then
  assertThat(result.size()).isEqualTo(2);
  }
}
```

- given - when - then 패턴으로 Test 진행
       
     1. given : 준비 - 테스트에 사용하는 변수,이름 정의
     
     2. when : 실행 - 실제로 기능 test 진행
     
     3. then :  검증 - 예상한 값과 실행 통해 나온 값을 검증

- @AfterEach

- 사용 이유  

: Test code의 장점 - 각각의 test별 뿐 아니라 전체를 한번에 test가 가능

→ 이 때, 한번에 여러개를 test하다 보면 메모리 DB에 직전 테스트 결과가 남아 이후 test할때 오류 가능성 

→ 따라서 한 test 가 끝날 때마다 공용공간의 메모리 DB에 저장된 데이터를 삭제하는 기능을 실행

- 테스트는 각각 독립적으로 실행되어야 한다. 테스트 순서가 서로 의존관계를 가지면 안된다.
- MemoryMemberRepository에 clearStore method 추가 Test code에 @AfterEach 코드 작성

```jsx
public void clearStore(){
	store.clear();
}
```

## 4. **회원 서비스 개발**

회원 서비스 

1. 회원 등록(가입) 
2. 회원 조회 

```jsx
package hello.hellospring.service;
import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import java.util.List;
import java.util.Optional;

public class MemberService {
 private final MemberRepository memberRepository = new MemoryMemberRepository();

 //회원가입
 public Long join(Member member) {
	 validateDuplicateMember(member); //중복 회원 검증
	 memberRepository.save(member); //중복 회원이 아니라면 저장 
	 return member.getId();
	 } 

 private void validateDuplicateMember(Member member) {
	 memberRepository.findByName(member.getName())
					 .ifPresent(m -> {
							throw new IllegalStateException("이미 존재하는 회원입니다.");
					 });
	 }

 // 전체 회원 조회
 public List<Member> findMembers() {
	 return memberRepository.findAll();
	 }

 // 해당 ID 가진 회원 조회 
 public Optional<Member> findOne(Long memberId){
    return memberRepository.findById(memberId);
    }
}
```

- 리포지토리는 save,findAll 등과 같이 데이터를 DB에 넣고 빼는 단순한 동작 느낌.
- 서비스 클래스는 join, findMembers 와 같이 비즈니스에 가까운 용어를 쓴다.
   
   → 담당하는 기능에 해당하는 비즈니스랑 맞게 매칭할 수 있는 용어를 선택하는 것도 중요 
      
   → 직관적으로 어떤 기능을 하는지 알 수 있기 때문

## 5. **회원 서비스 테스트**

- 단축키 :  ctrl + shift + t - test case 쉽게 만들 수 있음
- 단축키 : alt + enter - static method / 멤버 변수 import
- 단축키 : shift + f10 ( ios - ctrl + R) 이전에 실행을 그대로 다시 진행

```jsx
package hello.hellospring.service;
import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemoryMemberRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;

class MemberServiceTest {
 MemberService memberService;
 MemoryMemberRepository memberRepository;

 @BeforeEach
 public void beforeEach() {
	 memberRepository = new MemoryMemberRepository();
	 memberService = new MemberService(memberRepository);
	 }

 @AfterEach
 public void afterEach() {
	 memberRepository.clearStore();
	 }

 @Test 
 public void 회원가입() throws Exception {
  //Given
  Member member = new Member();
  member.setName("hello");
 
  //When
  Long saveId = memberService.join(member);
 
  //Then
  Member findMember = memberRepository.findById(saveId).get();
  assertThat(member.getName()).isEqualTo(findMember.getName());
 }

//test는 예외상황을 체크하는 것에 큰 의미 
  @Test
  public void 중복_회원_예외() throws Exception {
   //Given
   Member member1 = new Member();
   member1.setName("spring");
	 
	 Member member2 = new Member();
	 member2.setName("spring");

   //When
  memberService.join(member1);
  IllegalStateException e = assertThrows(IllegalStateException.class,
  () -> memberService.join(member2));// validate에서 걸려서 예외가 발생해야 한다.
  assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
	//Try-catch 대신 간결하고 좋은 코드 사용
  /**
  try{
      memberService.join(member2);
      fail();
   }catch (IllegalStateException e){
      assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
   }
  **/
  }
}
```

- 기존에는 MemberService class와 test class에서 각각 memoryMemberRepository를 생성

```jsx
public class MemberService {
	 private final MemberRepository memberRepository = new MemoryMemberRepository();
}
```

```jsx
 class MemberServiceTest{
	 MemberService memberService = new MemberService();
	 MemoryMemberRepository memberRepository = new MemoryMemberRepository();   
}
```

  → 각각 객체를 생성하면 다른 인스턴스이므로 들어있는 내용이 달라 정확한 Test가 안 될수도 있음.

- 해결방법

```jsx
public class MemberService { 
	private final MemberRepository memberRepository;
 
  public MemberService(MemberRepository memberRepository) {
		 this.memberRepository = memberRepository;
  }
 ...
}
```

```jsx
class MemberServiceTest {
 MemberService memberService;
 MemoryMemberRepository memberRepository;

 @BeforeEach
 public void beforeEach() {
	 memberRepository = new MemoryMemberRepository();
	 memberService = new MemberService(memberRepository);
 }
...
}
```

 → Test는 각각 독립적으로 실행되어야 하기 때문에 @BeforeEach 코드 이용해서 각 test실행 전에 MemoryMemberRepository 객체 만들고 MemberService에 넣어줌. 

- MemberService 에서 보면 MemoryMemberRepository 객체를 외부에서 넣어줌

     → DI(Dependency Injection) 가능하게 해줌
