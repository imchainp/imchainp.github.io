# ✏️서두

- 컨트롤러의 requestBody에 @Valid 어노테이션을 붙이면, 어떻게 마법처럼 객체의 유효성을 검증할까요?
- @Email 과 같은 jakarta validation은 어떻게 이루어질까요? 구현체를 알아봐요.
- @Valid 를 사용하려면 왜 build.gradle에 `implementation 'org.springframework.boot:spring-boot-starter-validation'` 을 추가해줘야 할까요?

---

# 📄요약

1. Spring 의 컨트롤러에서 @Valid 를 사용하려면 `implementation 'org.springframework.boot:spring-boot-starter-validation'` 붙여줘야 합니다.

   ![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled.png)

  - `[org.springframework.boot:spring-boot-starter-validation` 프로젝트](https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-starters/spring-boot-starter-validation/build.gradle)에 가보면 위와 같이 org.hibernate.validator:hibernate-validator 를 api로 가져오고 있습니다. @Valid의 경우,  `jakarta.validation.Valid` 인데, [org.hibernate.validator:hibernate-validator](https://github.com/hibernate/hibernate-validator/blob/main/engine/pom.xml#L39-L42) 는 `jakarta.validation` 패키지를 가져오고 있습니다. 따라서 @Valid 어노테이션을 사용하기 위해 @Valid를 사용하는 모듈의 build.gradle에`implementation 'org.springframework.boot:spring-boot-starter-validation'` 를 붙여줍니다.
  - `org.springframework.boot:spring-boot-starter-validation` 을 implementation 으로 가져오는 제일 중요한 이유는, `ValidatorImpl`**를 Spring 컨텍스트에 bean 으로 등록하기 위함입니다.**
    - [org.hibernate.validator:hibernate-validator](https://github.com/hibernate/hibernate-validator/blob/main/engine/pom.xml#L39-L42) 에 대해 기본적인 bean(`ValidatorImpl`) 등록이 어떤 것이 이루어지는지, **이건 따로 포스팅 예정입니다. (org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration.EnableWebMvcConfiguration#mvcValidator)**
2. @RequestBody에 대한 @Valid 어노테이션에 대한 처리는 Spring-webmvc  모듈의 `RequestResponseBodyMethsodProcessor` 의 `validateIfApplicable` 함수를 통해 처리됩니다.
  - `RequestResponseBodyMethodProcessor` 는  Spring-webmvc 에서 `HandlerMethodArgumentResolver` 에 기본적으로 등록(add)됩니다.
  - Spring  서버에 요청이 들어오면, handlerMethod 의 argument에 대해 resolving 하는 과정에서 `RequestResponseBodyMethodProcessor` 의 `resolveArgument` 가 실행됩니다. 그 과정에서 유효성 검사를 하는 `validateIfApplicable` 가 실행됩니다.
3. @Email 과 같은 jakarta.validation 은 위 2번 과정에서 EmailValidator 를 통해 유효성 검사를 진행하게 됩니다. 구현체는 아래와 같습니다.

   ![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%201.png)

   ![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%202.png)

---

# 👨‍💻@Valid 어노테이션이 어떻게 동작하는지 디버깅 과정

### 💻프로젝트 코드

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%203.png)
![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%204.png)

프로젝트는 우리가 흔히 아는 RestConroller 에 컨트롤러 메소드를 하나 @PostMapping 해주어 요청을 받게 했습니다. 그리고 파라미터 어노테이션으로, POST 요청 requestBody의 유효성 검사를 위해  @Valid 와 @RequestBody 를 붙여주었습니다.

### 🔍 `/api/v1/validate-request-body` 요청 후 디버깅

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%205.png)

디버깅 과정은 위 요청을 보내면서 시작이 됩니다. 스프링을 공부하셨다면 다들 한번쯤은 들어보셨을,`DispatcherServlet` 부터 시작됩니다. 디버깅에 앞서서 전체적인 큰 그림을 말씀르드리자면, 요청을 보내고 나서 `doDispatch` 는 handler 맵핑을 하게 되고, argument resolving 하는 과정에서 validation이 진행됩니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%206.png)

위 코드에서 `mv = ha.handle(processedRequest, response, mappedHandler.getHandler());` 하는 부분부터 들어갑니다. 이후 작업으로 argument를 resolving 하는 과정이 이어지는데, `RequestResponseBodyMethodProcessor` 라는 클래스의 `resolveArgument` 함수로 진입하게 됩니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%207.png)

위 코드에서 실질적으로 validation 진입점은 `validateIfApplicable` 이라는 함수입니다. 사전적인 의미로는 적용할 수 있는지 확인(검증)한다 정도인데요. 여기를 조금 더 파보도록 하겠습니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%208.png)

우리는 디버깅을 위해 POST `/api/v1/validate-request-body`  요청으로 validateTest 라는 메소드를 호출하였습니다. 그래서 TestConroller의 validateTest 메소드의 parameter(`@RequestBody @Valid body: ValidationTestRequest`) 의 어노테이션이 무엇이 있는지 확인을 하게 됩니다. 어노테이션이 2개 있으므로, 하나씩 순회하면서 validate 하는 식으로 이루어집니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%209.png)

위 코드에서 @Valid 에 대해 validationHint 가 있는지 확인하게 되는데요. `ValidationAnnotationUtils.*determineValidationHints*(ann)` 가 뭐하는 녀석인지 조금 더 살펴봐야겠습니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2010.png)

오 신기하게도 Validated 와 Valid 어노테이션에 대해서 모두 체크를 하네요. 우선 저희의 경우, 컨트롤러 메소드(validateTest) parameter에 Valid 어노테이션만 걸었으니, `if ("jakarta.validation.Valid".equals(annotationType.getName())) {}` 이쪽 블럭으로 빠지네요. `EMPTY_OBJECT_ARRAY` 는 `new Object[0];` 입니다. 빈 객체를 가지고 무엇을 할까요?

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2011.png)

다시 돌아와서 validationHints 가 null 이 아닌지 체크하는 구문에서 true를 반환하네요. (개인적인 의견으로  반환타입이 Optional 이면 어땠을까라는 생각이 드네요) 여러 케이스를 커버하려다보니 이렇게 구성한 것 같은데, 우선 그 후 진행될 binder.validate(validationHints) 를 봅시다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2012.png)

위 (DataBinder)binder.validate(validationHints) 는 getTarget() 이라는 함수로, DataBinder 내의 멤버변수인 target을 가져오는데, 이건 우리가 보낸 requestBody 객체를 참조하고 있네요.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2013.png)

맥락을 이어서 조금 더 진행해보면, getBindingResult() 라는 함수로 DataBinder 내에서 binding한 결과값을 가져오고 있습니다. for 문 위에 보시면 “binding 한 결과값과 일치하는지 각 validator 를 호출해본다.” 라고 주석이 되어 있네요.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2014.png)

DataBinder 클래스 내 ****getValidatorsToApply를 보시면, 이런 식으로 본인의 validator 를 결국 반환해주는 것을 확인할 수 있습니다. 다시 DataBinder 로 돌아와서, validation 을 어떻게 하는지 살펴보겠습니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2015.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2016.png)

이전에 넘겨받은 validationHint(EMPTY Object 배열 넘겨주신 것 기억하시죠?) 가 empty인지 확인하는 조건을 확인합니다. empty object 배열(length가 0)이기 때문에 **이 조건에서는 false 가 됩니다.**  getValidatorsToApply() 로 넘겨받은 Validator(ValidationAdapter) 에 대해 SmartValidator 인지 확인하는데요.  ValidationAdapter의 경우, SmartValidator를 구현하고 있기 때문에 **이 조건문은 true** 입니다. 아쉽지만 선행 조건의 false로 인해 else if 문으로 빠지게 됩니다.

자 이제 그러면, `validator.validate(target, bindingResult);` 내에서 진짜 validation 이 일어나겠네요.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2017.png)

`validator.validate()`를 파고파고 들어가보면, `SpringValidatorAdapter` 의 validate 함수가 등장하는데요. 여기에서 `this.targetValidator.validate(target)` 를 수행하고, `processConstraintViolations` 을 수행하게 됩니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2018.png)

`this.targetValidator.validate(target)` 를 수행하는 과정입니다. 살펴보기에는 복잡하고, 바로 함수 내 `validateInContext` 로 가보겠습니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2019.png)

`validateInContext` 메소드 위 주석을 보시면, validationContext 는 global 한 validation context인 것을 확인할 수 있고, valueContext 는 현재 validation context인 것을 확인하실 수 있습니다. 자세히 살펴보지는 못했지만 validation 한 결과를 global 한 validationContext 에 기록할 것 같습니다. 다시 돌아와, 이후  `ValidatorImpl.validateConstraintsForCurrentGroup` 에 한번 빠져보겠습니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2020.png)

`validateConstraintsForCurrentGroup` 을 살펴보면, 조건문에 의해 `validateConstraintsForDefaultGroup` 으로 빠지게 됩니다. 여기에서 무엇을 하냐면, 클래스의 계층적 구조(상속 구조)에 따라 조건을 검증하는 것으로 보입니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2021.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2022.png)

486 라인 뒤인 487 라인에서 `hostingBeanMetaData.getDirectMetaConstraints()` 을 호출하게 되는데요. `getDirectMetaConstraints()` 에서는 드디어 우리가 기다리고 기다리던 [@Email, @PastOrPresent, @Max] 어노테이션들을 찾을 수 있었네요. 이후 488~499 라인인 `validateConstraintsForSingleDefaultGroupElement` 메소드로 한번 들어가봅시다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2023.png)

오 이제 위 코드 보니깐 뭔가 506라인부터 metaConstraints(@Email, @PastOrPresent, @Max) 를 하나씩 순회하면서 검증하고 실패하는지 테스트하지 않을까라는 감이 오지 않으신가요? 우선 506번의 for문에서는 첫번째 원소로 Email 이네요. 그래서 `validateMetaConstraint` 는 email 이라는 metaConstraint 를 들고 들어갑니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2024.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2025.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2026.png)

`isValidationRequired` 에서는 validation이 필요한 것인지 판단하겠네요. 디버깅을 해보시면 validation이 필요한 것으로 true를 반환합니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2027.png)

자 정말 먼 길 떠나왔는데요. `metaConstraint.validateConstraint( validationContext, valueContext );` 에서 이제 email이라는 metaConstraint에 대해 validation을 진행하게 됩니다!

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2028.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2029.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2030.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2031.png)

`MetaConstraint` 의 `validateConstraint` 는 `doValidateConstraint` 를 호출하게 되고, `constraintTree.validateConstraints( executionContext, valueContext );` 를 또 다시 호출하게 됩니다. 여기에서 `validateConstraints` 를 하게 되고, `validateSingleConstraint` 를 또 호출합니다. (여기까지의 과정은 `SimpleConstraintTree` 의 66번 라인 break point 거시면 됩니다 😇)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2032.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2033.png)

자, 이제 찐찐최종 validation 입니다. 우선 위 코드의 175번 라인의 `ConstraintValidator<A, V> validator`를 보시면 EmailValidator로 되어 있습니다. 이 EmailValidator는 바로 [윗윗 사진](https://www.notion.so/Spring-RequestBody-Valid-6915c4b99d5e4b3c81c5abf6eebcc6a8?pvs=21)(`SimpleConstraintTree`)의 58번 라인(`getInitializedConstraintValidator(validationContext, valueContext)`)에서 찾아온 것입니다.

validatedValue를 보시면 valueContext에서 현재의 validated 된 값을 가져오고, 우리가 request body의  email 필드로 보내줬던 “ddddddddd” 를 가져오게 됩니다. 이제 180번 라인인 `isValid = validator.isValid( validatedValue, constraintValidatorContext );` 로 들어가봅시다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%201.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%202.png)

위와 같이 `boolean isValid = super.isValid( value, context );` 를 호출하게 되고, 이메일에 대한 validation을 진행하게 되네요. 드디어 Validator에서 유효한지 체크하는 isValid까지 오게 됐습니다! 저희가 보내준 ddddddddd 는 ‘@’ 가 없어서 valid 하지 않으므로 false 를 반환하네요.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2034.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2035.png)

결국 `SimpleConstraintTree` 의 `validateSingleConstraint` 는 Optional.of(constraintValidatorContext) 를 반환하게 되네요. 또한 validation에 실패하게 되므로 `violatedConstraintValidatorContexts` 에도 추가됩니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2036.png)

이후 여러 과정을 거쳐 원점인 `RequestResponseBodyMethodProcessor` 로 돌아오게 되고, 위 142번 조건문을 통해 binder의 bindingResult가 오류가 있었는지(hasErrors()), binding 과정에서 Exception이 필요한지 체크합니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2037.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2038.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2039.png)

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2040.png)

그래서 결국 `RequestResponseBodyMethodProcessor` 의 143 라인인 `throw new MethodArgumentNotValidException(parameter, binder.getBindingResult());` 을 수행하게 되네요. 이건 이제 DispatcherServlet catch 하여 이후 처리가 되고 아래와 같이 오류 응답을 받을 수 있습니다.

![Untitled](../assets%2Fimg%2F2024-02-16-%5BSpring%5D%20%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC%EC%9D%98%20%40RequestBody%EC%97%90%20%EB%8C%80%ED%95%9C%20%40Valid%EB%8A%94%20%EC%96%B4%EB%96%A4%20%EB%B0%A9%EC%8B%9D%EC%9C%BC%EB%A1%9C%20%EB%8F%99%EC%9E%91%ED%95%A0%EA%B9%8C%3F%2FUntitled%2041.png)

---

# ✅마무리

오늘은 @Valid 어노테이션이 어떻게 동작하는지에 대해 알아보았습니다. 간단한 validation도 여러 가지 객체가 관여하여 context에 기록도 하고, 매개변수를 넘겨주는 과정들이 인상 깊었던 것 같습니다.

또, 아무래도 Spring 자체가 큰 프로젝트다 보니 여러 가지 경우를 처리하기 위해 디버깅하는 depth 가 정말 깊었습니다. validation 하나 하는데 이렇게나 많은 객체가 필요하다니.. 너무 복잡한 것 아닐까라는 생각도 있지만, 여러 가지 constraint 를 조합하여 group으로 validation 하는 요구사항이나, global 컨텍스트에 현재 컨텍스트의 validation 결과를 기록하는 요구사항까지 처리하려면 어쩔 수 없었던 것 같습니다.

확실히 Spring은 디버깅 하면 할수록 알아가는 게 많은 것 같습니다. 회사에서 프로젝트를 진행할 때, @Valid 를 사용하기 위해 무의식적으로 사용했었는데요, 이번 기회에 조금 더 자세하게 알아갈 수 있어서 좋았습니다. 다음글은 [요약 부분에서 언급한 gradle 설정의 이유](https://www.notion.so/Spring-RequestBody-Valid-6915c4b99d5e4b3c81c5abf6eebcc6a8?pvs=21)에 대해 더 알아보려고 합니다.
