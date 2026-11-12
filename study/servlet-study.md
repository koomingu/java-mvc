# Servlet과 Servlet Container 학습 메모

학습 페이지: https://techcourse-lms-plus-web.woowahan.com/missions/46?step=420

## 읽기 전에 알아둘 개념

1. HTTP 요청과 응답

브라우저가 서버에 보내는 메시지가 요청이고, 서버가 돌려주는 메시지가 응답입니다. `/shared-counter`는 요청 경로입니다. 상태 코드 200은 정상 처리, 404는 요청한 대상을 찾지 못했다는 뜻입니다.

2. 서블릿과 톰캣

서블릿은 요청을 받아 응답을 만드는 자바 객체입니다. 톰캣은 요청을 받아 알맞은 서블릿을 실행하고, 객체의 생성과 초기화, 종료도 관리합니다. 이런 역할을 하는 프로그램을 서블릿 컨테이너라고 합니다. `@WebServlet`은 서블릿과 요청 경로를 연결합니다.

3. 서블릿의 생명주기

`init()`은 서블릿을 처음 사용할 때 준비하는 메서드입니다. `service()`는 요청마다 실행되고, `destroy()`는 서블릿을 종료할 때 실행됩니다. 이 메서드들은 톰캣이 호출합니다. 초기화는 설정에 따라 서버 시작 시점에 할 수도 있습니다.

4. 멤버 변수와 지역 변수

멤버 변수는 객체가 가지고 있는 값이라 메서드 실행이 끝나도 남습니다. 지역 변수는 메서드를 호출할 때마다 새로 생깁니다. 같은 서블릿 객체가 여러 요청을 처리하므로, 사용자별 정보를 멤버 변수에 넣으면 다른 요청과 섞일 수 있습니다. 여러 스레드가 공유 값을 동시에 바꾸면 일부 변경이 누락될 수도 있습니다.

5. 필터와 스프링 MVC

필터는 서블릿 실행 전후에 공통 작업을 합니다. `chain.doFilter()`를 호출하면 다음 필터나 서블릿으로 처리가 넘어가고, 호출이 끝나면 현재 필터로 돌아옵니다. 스프링 MVC에서는 `DispatcherServlet`이 요청을 받아 알맞은 컨트롤러를 실행합니다. Interceptor는 이 내부에서 컨트롤러 실행 전후를 처리합니다.

6. 문자 인코딩

네트워크는 문자를 바이트로 주고받습니다. UTF-8은 문자와 바이트를 변환하는 규칙 중 하나입니다. 이번 코드에서는 응답 인코딩을 지정하지 않으면 한글이 `???`로 바뀌었습니다. 필터에서 `chain.doFilter()`를 호출하기 전에 UTF-8을 지정해야 서블릿의 `getWriter()`가 해당 인코딩을 사용합니다. 요청 인코딩과 응답 인코딩은 별도로 설정합니다.

응답의 Content-Type과 charset을 클라이언트에 알리는 일도 필요합니다. 이번 필터는 문자 인코딩을 설정하고, 응답 형식은 각 서블릿에서 정하도록 두었습니다. 이번 학습 테스트의 HTTP 클라이언트는 charset 헤더가 없으면 UTF-8로 응답을 읽습니다.

## 코드 실행 결과

Java 21로 내장 톰캣을 띄워 실제 HTTP 요청을 보내는 `ServletTest`와 `FilterTest`를 실행했습니다. 기존 8080 포트가 사용 중이라 학습용으로 18080 포트를 지정했습니다. 두 테스트 클래스가 같은 포트를 쓰므로 순서대로 실행하도록 설정했습니다.

수정 전에는 sharedCounter의 세 번째 응답이 3, localCounter의 세 번째 응답이 1이었습니다. 한글 응답은 `???`였습니다. 카운터 예상값을 각각 3과 1로 바꾸고, 필터에서 요청과 응답 인코딩을 UTF-8로 설정한 뒤 테스트 3개가 모두 통과했습니다.

카운터 테스트의 로그에서는 서블릿마다 `init()` 한 번, 요청마다 `doFilter()`와 `service()`, 종료할 때 `destroy()` 한 번을 확인했습니다. FilterTest에서도 `doFilter()`가 `service()`보다 먼저 실행됐습니다. 필터에서 `chain.doFilter()` 뒤에 로그를 추가하면 서블릿 처리 후 필터로 돌아오는 시점도 확인할 수 있습니다.

프로젝트 루트에서 PowerShell로 다시 실행할 수 있습니다. JAVA_HOME은 이 컴퓨터에 설치된 Java 21 경로입니다.

```powershell
$env:JAVA_HOME='C:\Program Files\Java\jdk-21'
.\gradlew.bat '-Dstudy.http.port=18080' :study:test --tests servlet.com.example.ServletTest --tests servlet.com.example.FilterTest --console=plain
```

`study.http.port`를 생략하면 기본 포트는 8080입니다. IntelliJ에서 `ServletApplication.main()`을 직접 실행할 때도 VM 옵션에 `-Dstudy.http.port=18080`을 넣으면 같은 포트를 쓸 수 있습니다. 브라우저에서 `http://localhost:18080/shared-counter`와 `http://localhost:18080/local-counter`를 새로고침하며 값을 비교해 보세요.

최종 테스트 보고서는 `study/build/reports/tests/test/index.html`에 있습니다. 실행 로그는 `build/servlet-study-after.log`, 생명주기 로그는 `study/build/test-results/test`의 XML 파일에 있습니다. 테스트 결과 파일은 다음 실행 시 덮어써집니다.

## LMS에 저장한 답변

### init(), service(), destroy() 메서드가 언제 실행됐나요?

init()은 서블릿 객체가 생성된 뒤 첫 요청을 처리하기 전에 한 번 실행됐습니다. service()는 요청이 들어올 때마다 실행돼, 세 번 요청한 카운터 테스트에서는 세 번 호출됐습니다. destroy()는 테스트에서 톰캣을 종료할 때 한 번 실행됐습니다. 이 메서드들은 직접 호출하는 것이 아니라 서블릿 컨테이너인 톰캣이 호출합니다. 초기화 시점은 설정에 따라 서버 시작 시점이 될 수도 있습니다.

### testSharedCounter()와 testLocalCounter()의 결과가 다른 이유는 무엇인가요?

testSharedCounter()의 마지막 응답은 3입니다. sharedCounter는 서블릿 객체의 멤버 변수라 요청이 끝나도 값이 남습니다. 같은 객체가 세 요청을 처리하면서 값이 1, 2, 3으로 증가합니다.

testLocalCounter()의 마지막 응답은 1입니다. localCounter는 service() 안의 지역 변수라 호출할 때마다 0으로 새로 만들어지고, 한 번 증가한 1을 응답합니다.

이번 테스트는 요청을 순서대로 보낸 결과입니다. 여러 요청이 동시에 sharedCounter를 증가시키면 서로의 변경을 덮어써서 일부 증가가 누락될 수 있습니다.

### Filter와 Interceptor는 어떤 차이가 있을까요?

Filter는 서블릿 표준 기능으로, 서블릿 컨테이너가 실행합니다. DispatcherServlet에 들어가기 전과 처리가 끝난 뒤에 동작하며, 스프링 없이도 사용할 수 있습니다. doFilter()에서 chain.doFilter()를 호출하면 다음 필터나 서블릿으로 처리를 넘깁니다. 인코딩 설정이나 요청과 응답을 감싸서 바꾸는 처리에 적합합니다.

Interceptor는 스프링 MVC 기능으로, DispatcherServlet 내부에서 컨트롤러 실행 전후에 동작합니다. preHandle(), postHandle(), afterCompletion()을 사용하며, 실행할 컨트롤러 메서드 정보도 확인할 수 있습니다. 컨트롤러별 실행 시간 측정처럼 MVC 처리에 관련된 작업에 적합합니다.

## 참고한 공식 문서

응답 인코딩은 `getWriter()` 전에 지정해야 합니다. [ServletResponse 문서](https://docs.oracle.com/javaee/7/api/javax/servlet/ServletResponse.html)

Interceptor의 실행 시점과 메서드는 스프링 MVC 문서에서 확인할 수 있습니다. [Spring MVC Interception](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/handlermapping-interceptor.html)

서블릿의 생성과 생명주기는 서블릿 명세에 정의되어 있습니다. [Jakarta Servlet 6.1 명세](https://jakarta.ee/specifications/servlet/6.1/jakarta-servlet-spec-6.1.html)
