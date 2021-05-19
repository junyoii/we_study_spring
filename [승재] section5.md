# 회원 관리 예제 – 웹 MVC 개발

이 강의에서는 실질적으로 만든 로직들의 컨트롤러를 생성하고 **Thymeleaf** 엔진을 이용해서 화면에 뿌려주는 실습을 진행하였다.

## 회원 웹 기능 - 홈 화면 추가

우선 메인 페이지를 보여주기위해 HomeController를 만들어서 루트 url에 home.html을 매핑시켜준다. 방식은 당연히 **GetMapping**으로 진행한다.

### HomeController

```java
@Controller
public class HomeController {
    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

## 회원 웹 기능 - 회원 등록 화면 추가, 조회

회원 등록을 받는 것은 form을 통해서 받을 것이다. 그리고 해당 form에서 받는 데이터를 편하게 받기 위해서 **MemberForm**을 생성하여 준다. 그리고 form에서 받은 데이터를 post로 보내주고 그 데이터가 MemberForm에 매핑된다. 그리고 memberName값이 들어간 MemberForm 인스턴스가 인자로 create 함수가 실행되고 멤버가 추가되게 된다. 여기서 헷갈리기 쉬운 점은 form에서 input값이 넘어올때 매핑 되는 기준은 id가 아닌 **name**이라는 것이다.

이렇게 회원을 등록 후에는 List형식으로 member 목록을 받은 뒤에 **memberList.html**로 넘겨주고 **Thymeleaf** 문법을 통해서 안에 항목들을 화면에 표시해주게 된다.

### MemberForm

```java
public class MemberForm {
    private String memberName;

    public String getMemberName() {
        return memberName;
    }

    public void setMemberName(String memberName) {
        this.memberName = memberName;
    }
}
```

### MemberController

```java
@Controller
public class MemberController {
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
    
    @GetMapping("/members/new")
    public String createForm() {
        return "members/createMemberForm";
    }

    @PostMapping("/members/new")
    public String create(MemberForm form) {
        Member member = new Member();
        member.setName(form.getMemberName());
        memberService.join(member);

        return "redirect:/";
    }

    @GetMapping("/members")
    public String list(Model model) {
        List<Member> members = memberService.findMembers();
        model.addAttribute("members", members);
        return "members/memberList";
    }
}
```

## static, final (여기는 제가 강의를 듣다가  궁금해서 추가적 공부 후 작성한 내용입니다. 아마 민경이가 한 내용 부분일 겁니다.)

자 그럼 몇가지 의문이 들 수 있습니다. 물론 저만 그럴 수도 있긴해요...ㅎ static과 final에 대한 혼란이 옵니다.

#### MemberService
```java
private final MemberRepository memberRepository;
```

#### MemoryMemberRepository
```java
private static Map<Long, Member> store = new HashMap<>();
```

대체 뭔 차이일까요? 일단 기본적인 **static, final**에 대해서 간단히 알아보겠습니다.
우선 **final은 자바에서 해당 entity가 오로지 한 번 할당될 수 있음을 의미하는 키워드**입니다. 클래스에 사용하면 클래스의 상속을 막아주는 기능도 하죠.

그리고 **static**은 해당 데이터의 메모리 할당을 컴파일 시간에 할 것임을 의미합니다. 클래스 내에서 멤버 변수를 선언할 경우 클래스가 컴파일시 로딩될 때 해당 멤버 변수에 대한 메모리 할당이 된다고 생각해도 됩니다.
자 그럼

```java
Class A {
	private static final int AGE = 24;
}
```

이런 식의 코드를 java를 사용하시는 분은 자주 봤을 것입니다. 이것의 의미는 A클래스를 로딩시 일단 AGE라는 변수를 24로 초기화하여 메모리에 할당후 다시 해당 변수를 바꿀 수 없다는 의미입니다.
그럼 위에서

```java
private static Map<Long, Member> store = new HashMap<>();
```

이것의 의미는? 그렇습니다. store를 MemoryMemberRepository 클래스를 로딩할 때 메모리를 할당한다는 것입니다. 그렇다면 굳이 store에 대한 내용을 바꾸지 않으니(데이터를 넣고 삭제하고 빼고가 아닌 store 그 자체를 바꾸지 않는 다는 것입니다.) final을 붙혀도 되지않냐고요? 네 맞습니다. 사실 붙혀야합니다..ㅎㅎ ㅋㅋㅋㅋ IDE에서도 final을 안 사용할경우 노란색 줄로 아마 final 필드일 것이라고 알려줍니다..ㅎㅎ 저는 해당 실습에 final을 추가해놨습니다.

추가적으로 **DI를 사용할 경우 static과 혼용은 최대한 피하는 것이 좋습니다.** Spring에서 DI를 사용한다는 것은 IOC 컨테이너에 해당 인스턴스에 생성과 소멸에 대한 권한을 위임하는 것입니다. 근데 static의 경우 해당 권한 밖입니다. static영역에서의 생성과 소멸은 IOC컨테이너의 영역 밖입니다. 해당 영역에서의 생성과 소멸은 프로그램의 로딩시 생성되고 종료시 삭제되기 때문에 여러 문제가 발생할 수 있는 것이죠.

### Ref
https://djkeh.github.io/articles/Why-should-final-member-variables-be-conventionally-static-in-Java-kor/

https://madplay.github.io/post/spring-framework-static-field-injection
