---
title: Clean Code Study (Ch.3 Function)
date: 2021-07-19 22:07:43
category: CleanCode
thumbnail: { '../../assets/clean-code-thumbnail.jpeg' }
draft: false
---

![thumbnail](../../assets/clean-code-thumbnail.jpeg)

회사에서 '클린코드' 책 스터디를 진행하고 있어 공부한 내용을 정리해보려고 한다.

# 03.함수
- 어떤 프로그램이든 가장 기본적인 단위가 함수
- 이번 장은 함수를 잘 만드는 방법에 대해 소개

### 작게 만들어라
- 80년대에는 함수가 한 화면을 넘어가면 안된다고 말했다.
- 요즘은 좋은 모니터를 사용하는 관계로 가로 150자, 세로 100줄을 넘어가면 안된다.
- if 문/else 문/while 문 등에 들어가는 블록은 한줄을 추천(들여쓰기)
- 즉, 중첩구조가 생길만큼 함수가 커져서는 안된다. (들여쓰기 수준은 1단이나 2단을 넘어서면 안됨)

### 한 가지만 해라
- 함수는 한가지를 해야한다. 그 한가지를 잘 해야 한다. 그 한가지만을 해야 한다.
- 함수가 몇가지 이를 하는지 판단하기 위해서 TO 문단으로 기술할 수 있다.  
  (TO 문단은 LOGO 언어에서 사용하는 키워드 라고 함. 루비나 파이썬에서 사용하는 'def'와 똑같다고 보면 된다고 함)  
  ex) TO RenderPageWithSetupsAndTeardowns, 페이지가 테스트 페이지인지 확인한 후 테스트 페이지라면 설정 페이지와 해제 페이지를 넣는다. 테스트 페이지든 아니든 페이지를 HTML로 렌더링한다.
- 위 예시에서 추상화 수준이 하나인 단계만 수행한다면 그 함수는 한 가지 작업만 한다는 것.
- 함수가 한가지만 하는지 판단하는 추가적인 방법으로, 단순히 다른 표현이 아니라 의미 있는 이름으로 다른 함수를 추출할 수 있다면 그 함수는 여러 작업을 하는 셈이다.

### 함수 당 추상화 수준은 하나로
- 함수가 확실히 '한 가지' 작업만 하려면 함수 내 모든 문장의 추상화 수준이 동일해야 한다.
- 추상화 수준은 높음, 중간, 낮음으로 평가할 수 있다.
- 한 함수 내에 추상화 수준을 섞으면 코드를 읽는 사람이 헷갈린다. (추상화 수준이 높음이면 높음을 유지, 낮음이면 낮음을 유지해야 한다.)
- 코드를 위에서 아래로 읽으면 함수 추상화 수준이 한번에 한 단계씩 낮아지는데, 이를 내려가기 규칙이라고 부른다.
- 추상화 수준이 하나인 함수를 구현하기란 쉽지 않기 때문에, 중요한 핵심은 짧으면서도 '한 가지' 만 하는 함수다.

### Switch 문
- switch 문은 작게 만들기 어려운것이 사실
- switch 문은 본질적으로 n가지를 처리하는 문법이기 때문에 한 가지 작업만 하는 switch 문을 만들기는 어렵다.
- switch 문을 저차원 클래스에 숨기고 절대로 반복하지 않는 방법은 있다.
```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
	switch (e.type) { 
		case COMMISSIONED:
			return calculateCommissionedPay(e); 
		case HOURLY:
			return calculateHourlyPay(e); 
		case SALARIED:
			return calculateSalariedPay(e); 
		default:
			throw new InvalidEmployeeType(e.type); 
	}
}
```
- 위 함수는 몇가지 문제가 있다. 함수가 길고, 새 직원 유형을 추가하면 더 길어진다.
- '한 가지' 작업만 수행하지 않고, SRP(단일 책임 원칙) 을 위반한다. 단일 책임 원칙이란 어떤 클래스나 모듈이 단 하나의 변경하려는 이유만을 가져야 하는데 위 코드는 변경할 이유가 여럿이기 때문에 SRP를 위반한다.
- OCP(개방-패쇄 원칙) 도 위반하는데, OCP는 소프트웨어 개체(클래스, 모듈, 함수 등등)는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다'는 프로그래밍 원칙이다. 위 코드는 새 직원 유형을 추가할 때마다 코드를 변경하기 때문에 OCP를 위반한다.
- 위 코드는 switch 문을 추상 팩토리 안에 꽁꽁 숨겨서 해결 가능하다.

```java
public abstract class Employee {
	public abstract boolean isPayday();
	public abstract Money calculatePay();
	public abstract void deliverPay(Money pay);
}
-----------------
public interface EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType; 
}
-----------------
public class EmployeeFactoryImpl implements EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
		switch (r.type) {
			case COMMISSIONED:
				return new CommissionedEmployee(r) ;
			case HOURLY:
				return new HourlyEmployee(r);
			case SALARIED:
				return new SalariedEmploye(r);
			default:
				throw new InvalidEmployeeType(r.type);
		} 
	}
}
```
>추상 팩토리란?  
서로 관련이 있는 객체들을 통째로 묶어서 팩토리 클래스로 만들고, 이들 팩토리를 조건에 따라 생성하도록 다시 팩토리를 만들어서 객체를 생성하는 패턴  
즉, 관련성 있는 여러 종류의 객체를 일관된 방식으로 생성하는 경우에 유용하다.  
추상 팩토리는 싱글톤 패턴, 팩토리 매서드 패턴을 사용함  
ex) 삼성 컴퓨터 객체 공장, LG 컴퓨터 객체 공장

### 서술적인 이름을 사용하라
- testableHtml 보다 SetupTeardownIncluder.render 와 같이 하는 일을 좀 더 잘 표현하도록 이름을 짓는것이 좋다.
- 함수가 작고 단순할수록 서술적인 이름을 고르기도 쉬워진다.
- 이름이 길어도 괜찮음. 길고 서술적인 이름이 짧고 어려운 이름보다 좋다. 주석보다 좋다.
- IDE를 사용해 이름 바꾸기도 좋다.
- 서술적인 이름을 사용하면 개발자 머릿속에서도 설계가 뚜렷해지므로 코드를 개선하기도 쉬워진다.
- 모듈 내에서 함수 이름은 같은 문구,명사,동사를 사용한다.  
  ex) includeSetupAndTeardownPages, includeSetupPages, includeSuiteSetupPage, includeSetupPage 등 문체가 비슷하면 이야기를 순차적으로 풀어가기도 쉬움.

### 함수 인수
- 함수에서 이상적인 인수 개수는 0개(무항), 그 다음으로 1개(단항), 2개(이항) 순이고, 3개(삼항) 이상부터는 가능한 피하는 편이 좋다.
- includeSetupPageInfo(new PageContent) 보다 includeSetupPage() 가 더 이해하기 쉽다.
- 테스트 코드를 작성하는 관점에서도 인수가 없는 함수에 대해 테스트 케이스를 작성하는 것이 더 쉽다.

#### 많이 쓰는 단항 형식
- 함수에 인수 1개를 넘기는 이유로 가장 흔한 두 가지 경우가 있다.
- boolean fileExists("MyFile") 이 좋은 예
- InputStream fileOpen("MyFile") 은 String 형의 파일 이름을 InputStream 으로 변환하는 좋은 예
- passwordAttemptFailedNtimes(int attempts) 와 같은 이벤트 함수는 유용한 단항 함수 형식 이벤트
- 위 경우를 제외하고는 단항 함수는 가급적 피하는 것이 좋다.
- void includeSetupPageInto(StringBuffer pageText)와 같이 변환 함수에서 출력 인수를 사용하면 혼란을 일으킴

#### 플래그 인수
- 함수에 부울 값을 인수로 넘기는 것은 정말로 끔찍
- 플래그가 참이면 이걸 하고 거짓이면 저걸 한다는, 즉 함수가 여러 가지를 한꺼번에 처리한다고 대놓고 공표하는셈

#### 이항 함수
- 인수가 2개인 함수는 인수가 1개인 함수보다 이해하기 어렵다.
- wirteField(name) 이 writeField(outputStream, name) 보다 이해하기 쉬운 법
- 물론 이항 함수가 적절한 경우도 있는데, Point p = new Point(0, 0)가 좋은 예다. 인수 2개가 한 값을 표현하는 요소이다.
- 이항 함수가 무조건 나쁘다는 소리는 아니지만, 그만큼 위험이 따른다는 사실을 인지하고 최대한 단항 함수로 바꾸도록 애써야 한다.

#### 삼항 함수
- 당연하지만, 인수가 3개인 함수가 인수가 2개인 함수보다 훨씬 더 이해하기 어렵다.
- `assertEquals(message, expected, actual)` 는 첫 인수가 expected 라고 예상되어 주춤하게 되지만, `assertEquals(1.0, amount, .001)` 은 부동소수점 비교가 상대적이라는 사실을 인지할 수 있어 그리 음험하지 않은 삼항함수

#### 인수 객체
- 인수가 2~3개 필요하다면 일부를 독자적인 클래스 변수로 선언할 생각을 해보자.
- `Circle makecircle(double x, double y, double radius)` 와 `Circle makeCircle(Point center, double radius)` 와 같이 x, y를 묶어 넘기려면 결국 이름을 붙여야 하므로 개념을 표현해줄 수 있다.

#### 인수 목록
- 때로는 인수 개수가 가변적인 함수도 필요하다. `String.format`메서드가 좋은 예시
- `String.format("%s worked %.2f hours.", name, hours) 와 같이 사용하지만, 가변 인수 전부를 List 형 인수 하나로 취급할 수 있다.
- 실제로 `String.format` 선언부를 살펴보면 `public String format(String format, object... args)` 로 되어 있다.

#### 동사와 키워드
- 인수의 순서를 제대로 표현하려 할때도 좋은 함수 이름은 필수
- 함수와 인수가 동사/명사 쌍을 이루면 곧바로 이해하기 쉬움 `ex)write(name)`
- `writeField(name)` 처럼 사용하면 이름(name)이 필드(field) 라는 사실이 분명히 드러날 수 있다.
- `assertEquals` 보다 `assertExpectedEqualsActual(expected, actual)` 처럼 표현하면 인수 순서를 기억할 필요가 없다.

### 부수 효과를 일으키지 마라
- 부수 효과는 즉 거짓말, 함수에서 한 가지를 하겠다고 약속하고선 남몰래 다른 짓도 하는것.
```java
public class UserValidator {
  private Cryptographer cryptographer;

  public boolean checkPassword(String userName, String password) {
    User user = UserGateway.findByname(userName);
    if (user != User.NULL) {
      String codedPhrase = user.getPhraseEncodedByPassword();
      String phrase = cryptographer.decrypt(codedPhrase, password);
      if ("Valid Password".equals(phrase)) {
        Session.initialize();
        return true
      }
    }
  }
}
```

- 위 예제에서 Session.initialize() 호출이 부수 효과를 일으킨다.
- 함수 이름만 보면 checkPassword 함수로 세션을 초기화 한다는 사실은 드러나지 않는다. (함수 이름만 보고 호출하는 사용자는 사용자를 인증하면서 기존 세션 정보를 지워버릴 위험에 처함)
- checkPasswordAndInitializeSession 이라는 이름을 추천

#### 출력 인수
- `appendFooter(s)` 는 무언가에 s를 바닥글로 첨부할까? 아니면 s에 바닥글을 첨부할까? 이는 함수의 선언부를 찾아봐야 분명해진다.  
  `public void appendFooter(StringBuffer report)`
- 함수 선언부를 찾아보는 행위는 코드를 보다가 주춤하는 행위와 동급
- 인수 s가 출력 인수라는 사실은 분명하지만 함수 선언부를 찾아보고 나서야 알 수 있음
- 객체 지향 프로그래밍이 나오기 전에는 출력 인수가 불가피한 경우도 있었지만, 객체 지향 언어에서는 출력 인수를 사용할 필요가 거의 없다.
- 출력 인수로 사용하라고 설계한 변수가 바로 this 위 코드는 `report.appendFooter()` 와 같이 호출하는 방식이 좋다.

### 명령과 조회를 분리하라
- 함수는 뭔가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야 한다.
- `public boolean set(String attribute, String value)` 이 함수는 attribute 인 속성을 찾아 값을 value로 설정한 후 성공하면 true를 반환하고 실패하면 false를 반환하는 함수이다.  
  그래서 `if (set("username", "unclebob")) ...` 와 같이 괴상한 코드가 나온다.  
  "username"이 "unclebob"으로 설정되어 있는지 확인하는 코드인가? 아니면 "username"을 "unclebob"으로 설정하는 코드인가?
- 위 예시를 구현한 개발자는 "set"을 동사로 의도했지만 if문에 넣고 나니 형용사로 느껴진다.
- 해결책은 명령과 조회를 분리해 혼란을 애초에 뿌리뽑는 방법이다.
```java
if (attributeExists("username")) {
  setAttribute("username", "unclebob");
  .......
}
```

### 오류 코드보다 예외를 사용하라
- 명령 함수에서 오류 코드를 반환하는 방식은 명령/조회 분리 규칙을 미묘하게 위반한다.
- 자칫하면 if 문에서 명령을 표현식으로 사용하기 쉬운 탓
- `if (deletePage(page) == E_OK)` 이 코드는 동사/형용사 혼란을 일으키지 않는 대신 여러 단계로 중첩되는 코드를 야기시킴
- 오류 코드를 반환하면 호출자는 오류 코드를 곧바로 처리해야 한다는 문제에 부딪힘
```java
if (deletePage(page) == E_OK) {
	if (registry.deleteReference(page.name) == E_OK) {
		if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
			logger.log("page deleted");
		} else {
			logger.log("configKey not deleted");
		}
	} else {
		logger.log("deleteReference from registry failed"); 
	} 
} else {
	logger.log("delete failed"); return E_ERROR;
}
```
보다는
```java
try {
  deletePage(page);
  registry.deleteReference(page.name);
  configKeys.deleteKey(page.name.makeKey());
}
catch (Exception e) {
  logger.log(e.getMessage());
}
```
로 작성시 더 깔끔해진다.

#### Try/Catch 블록 뽑아내기
- try/catch 블록은 원래 추하기 때문에 별도 함수로 뽑아내는 편이 좋다.
```java
public void delete(Page page) {
	try {
		deletePageAndAllReferences(page);
  	} catch (Exception e) {
  		logError(e);
  	}
}

private void deletePageAndAllReferences(Page page) throws Exception { 
	deletePage(page);
	registry.deleteReference(page.name); 
	configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) { 
	logger.log(e.getMessage());
}

```
- 정상 동작과 어류 처리 동작을 위처럼 분리하면 코드를 이해하고 수정하기 더 쉬워진다.

#### 오류 처리도 한 가지 작업이다.
- 앞에서 말했듯이 함수는 '한 가지' 작업만 해야 하는데 오류 처리도 한 가지 작업에 속한다. 그러므로 오류 처리 하는 함수는 오류만을 처리해야 마땅하다.

#### Error.java 의존성 자석
- 오류 코드를 반환한다는 이야기는, 클래스든 열거형 변수든, 어디선가 오류 코드를 정의 한다는 뜻(ex. constants_pb의 Error)
```java
public enum Error { 
	OK,
	INVALID,
	NO_SUCH,
	LOCKED,
	OUT_OF_RESOURCES, 	
	WAITING_FOR_EVENT;
}
```
- 재컴파일/재배치가 번거롭기 때문에 Error enum을 새로 만드는 대신에 기존 오류 코드를 재사용하는 것이 좋다.

### 반복하지 마라
```java
public static String testableHtml(PageData pageData, boolean includeSuiteSetup) throws Exception {
        WikiPage wikiPage = pageData.getWikiPage();
        StringBuffer buffer = new StringBuffer();
        if (pageData.hasAttribute("Test")) {
            if (includeSuiteSetup) {
                WikiPage suiteSetup =
                    PageCrawlerImpl.getInheritedPage(SuiteResponder.SUITE_SETUP_NAME, wikiPage
                                                    );
                if (suiteSetup != null) {
                    WikiPagePath pagePath = suiteSetup.getPageCrawler().getFullPath(suiteSetup);
                    String pagePathName = PathParser.render(pagePath);
                    buffer.append("!include -setup .")
                          .append(pagePathName).append("\n");
                }
            }
            WikiPage setup = PageCrawlerImpl.getInheritedPage("SetUp", wikiPage);
            if (setup != null) {
                WikiPagePath setupPath =
                    wikiPage.getPageCrawler().getFullPath(setup);
                String setupPathName = PathParser.render(setupPath);
                buffer.append("!include -setup .")
                      .append(setupPathName).append("\n");
            }
        }
        buffer.append(pageData.getContent());
        if (pageData.hasAttribute("Test")) {
            WikiPage teardown = PageCrawlerImpl.getInheritedPage("TearDown", wikiPage);
            if (teardown != null) {
                WikiPagePath tearDownPath =
                    wikiPage.getPageCrawler().getFullPath(teardown);
                String tearDownPathName = PathParser.render(tearDownPath);
                buffer.append("\n")
                      .append("!include -teardown .").append(tearDownPathName).append("\n");
            }

        }
        if (includeSuiteSetup) {
            WikiPage suiteTeardown =
                PageCrawlerImpl.getInheritedPage(SuiteResponder.SUITE_TEARDOWN_NAME, wikiPage
                                                );
            if (suiteTeardown != null) {
                WikiPagePath pagePath = suiteTeardown.getPageCrawler().getFullPath(suiteTeardown);
                String pagePathName = PathParser.render(pagePath);
                buffer.append("!include -teardown .")
                      .append(pagePathName).append("\n");
            }
        }
    } 
    pageData.setContent(buffer.toString()); 
    return pageData.getHtml();
}

```
- 앞에 3-1 예제 코드에서 `SetUp, SuiteSetUp, TearDown, SuiteTearDown` 이 반복된다.
```java
public class SetupTeardownIncluder {

	private PageData pageData;
	private boolean isSuite;
	private WikiPage testPage;
	private StringBuffer	newPageContent;
	private PageCrawler pageCrawler;
	
	public static String render(PageData pageData) throws Exception {
		return render(pageData, false);
	}
	
	public static String render(PageData pageData, boolean isSuite) throws Exception {
		return new SetupTeardownIncluder(pageData).render(isSuite);
	}
	
	private SetupTeardownIncluder(PageData pageData) {
		this.pageData = pageData;
		testPage = pageData.getWikiPage();
		pageCrawler = testPage.getPageCrawler();
		newPageContent = new StringBuffer();
	}
	
	private String render(boolean isSuite) throws Exception {
		this.isSuite = isSuite;
		if (isTestPage())
			includeSetupAndTeardownPages();
		return pageData.getHtml();
	}
	
	private boolean isTestPage() throws Exception {
		return pageData.hasAttribute("Test");
	}
	
	private void includeSetupAndTeardownPages() throws Exception {
		includeSetupPages();
		includePageContent();
		includeTeardownPages();
		updatePageContent();
	}
	
	private void includeSuiteSetupPage() throws Exception {
		include(SuiteResponder.SUITE_SETUP_NAME, "-setup");		
	}
	
	private void includeSetupPage() throws Exception {
		include("Setup", "-setup");		
	}
	
	private void includePageContent() throws Exception {
		newPageContent.append(pageData.getContent());
	}
	
	private void includeTeardownPages() throws Excepton {
		includeTeardownPage();
		if (isSuite)
			includeSuiteTeardownPage();
	}
	
	private void includeTeardownPage() throws Exception {
		include("TearDown", "-teardown");
	}
	
	private void includeSuiteTeardownPage() throws Exception {
		include(SuiteResponder.SUITE_TEARDOWN_NAME, "-teardown");
	}
	
	private void updatePageContent() throws Exception {
		pageData.setContent(newPageContent.toString());
	}
	
	private void include(String pageName, String arg) throws Exception {
		WikiPage inheritedPage = findInheritedPage(pageName);
		if ( inheritedPage != null ) {
			String pagePathName = getPathNameForPage(inheritedPage);
			buildIncludeDirective(pagePathName, arg);
		}
	}
	
	private WikiPage findInheritedPage(String pageName) throws Exception {
		return PageCrawlerImpl.getInheritedPage(pageName, testPage);
	}
	
	private String getPathNameForPage(WikiPage page) throws Exception {
		WikiPagePath pagePath = pageCrawler.getFullPath(page);
		return PathParser.render(pagePath);
	}
	
	private void buildIncludeDirective(String pagePathName, String arg) {
		newPageContent
			.append("\n!include ")
			.append(arg)
			.append(" .")
			.append(pagePathName)
			.append("\n");
	}
}

```
- 3-7 코드에서 include 방법으로 중복을 없앤다.
- 코드를 중복하면 코드 길이가 늘어날 뿐 아니라 알고리즘/로직이 변하면 중복 되어 있는곳 모두 손을 봐야 한다. 게다가 어느 한곳이라도 빠뜨리면 오류가 발생할 확률도 높아진다.
- 객체 지향 프로그래밍에서는 코드를 부모 클래스로 몰아 중복을 없앤다.

### 구조적 프로그래밍
- 모든 함수와 함수 내 모든 블록에 입구와 출구가 하나만 존재해야 한다고 한다. 즉, return문이 하나여야 한다.
- 루프 안에서 break, continue를 사용해선 안되며 goto는 절대로 절대로 안된다.
- 함수가 작다면 간혹 return, break, continue를 여러 차례 사용해도 괜찮다. 오히려 때로는 단일 입/출구 규칙보다 의도를 표현하기 쉬워진다.

### 함수를 어떻게 짜죠?
- 소프트웨어를 짜는 행위는 여느 글짓기와 비슷하다. 논문이나 기사를 작성할 때 서투르고 중구난방의 어수선한 초안을 작성한다.
- 함수를 짤 대도 마찬가지. 처음에는 길고 복잡하고 들여쓰기 단계도 많고 중복된 루프도 많다. 인수 목록도 아주 길다. 이름은 즉흥적이도 코드는 중복된다. 하지만 그 서투른 코드를 빠짐없이 테스트 하는 단위 테스트 케이스도 만든다.
- 그런 후 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다. 메서드를 줄이고 순서를 바꾼다. 이 와중에도 단위 테스트는 항상 통과한다. (테스트 코드의 중요성...ㅠㅠ)
- 그래서 최정적으로 이 장에서 설명한 규칙을 따르는 함수가 얻어진다.

### 결론
- 함수는 그 언어에서 동사이며, 클래스는 명사이다. 프로그래밍의 기술은 언제나 언어 설계의 기술이다.
- 프로그래밍을 잘하는 프로그래머들은 시스템을 구현할 프로그램이 아니라 풀어갈 이야기로 여긴다.
- 프로그래밍 언어라는 수단을 사용해 좀 더 풍부하고 좀 더 표현력이 강한 언어를 만들어 이야기를 풀어가는 것
- 여기서 시스템에서 발생하는 모든 동작을 설명하는 함수 계층이 바로 그 언어에 속한다.
- 작성하는 함수가 분명하고 정확한 언어로 깔끔하게 같이 맞아떨어져야 이야기를 풀어가기가 쉬워진다는 사실을 기억하기 바란다.
