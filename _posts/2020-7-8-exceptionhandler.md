---
layout: post
title:  SpringBoot ExceptionHandling 
---
springboot에서 오류발생시 처리 하는 방법을 알아본다.  

java exception hierarchy

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7c48f813-6c5e-4a60-b86e-68bdd6b2b80d/exception-hierarchy-in-java.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7c48f813-6c5e-4a60-b86e-68bdd6b2b80d/exception-hierarchy-in-java.png)

### BasicErrorControll

스프링 부트 기본 에러처리 컨트롤러

application.properties에 정의된 기본설정(server.error)에 따라 응답 처리

오류 응답시 출력되는 정보 제어

```java
@Component
public class UnhandledExceptionAttributes extends DefaultErrorAttributes {

    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
	
    Map<String, Object> errorAttributes = super.getErrorAttributes(webRequest, includeStackTrace);

		// 아래와 같이 다양한 오류를 추가 할수 있음 
    if((Integer)errorAttributes.get("status") == 404){
	    errorAttributes.put("NOT FOUND" , String.format("[PATH] %s", errorAttributes.get("path")));
		}

    return errorAttributes;
} 
```

애플리케이션에서 발생하는 모든 에러 프로젝트 공통 포멧으로 정의 가능

### ExceptionHandler

Controller, RestController에서 발생되는 모든 예외를 처리 한다. 컨트롤러 별로 실행되기 떄문에 모든 컨트롤러에 정의 해줘야 한다.

```java
@RestController
public class testConotroller {

  // 에러 응답 코드 정의 
  @ResponseStatus(HttpStatus.NOT_FOUND)

  // 해당 컨트롤러에서 발생하는 예외(NullPointException)는 이 함수로 응답이 나가게 된다. 
	@ExceptionHandler(NullPointException.class)
	public String exception(){
		return "exception 발생";
	}
}
```

### ControllerAdvice

각 컨트롤러별로 예외처리를 하는게 아니라 컨트롤러 전체의 예외처리를 하는  annotation

filter에서 오류 발생시 해당 advice에서 처리되지 않는다.

```java
@RestControllerAdvice
public class ExceptionHandler{

  @AllArgsConstructor
  static class BindErrInfo {
      public String WHERE;
      public String FIELD;
      public Object INPUT;
      public String CAUSE;
  }

	@ExceptionHandler(customException.class)
	public String customException(){
    // exception 발생시 response body 출력 
    return commonExceptionResponse;
	}

  // DTO @Valid 바인딩 오류 처리 
	@ExceptionHandler(BindException.class)
  public Res handleHttpRequestBindException(BindException e) {
      ExceptionLogger.log(e);

      Res apiRes = res.error(ERR_INVALID_INPUT, "DTO 바인딩 오류");

      // 바인딩 에러 발생시 구체적인 에러내용을 리턴하기 위해서 처리 
      List<BindErrInfo> bindErrInfoList = new ArrayList<>();
      for (FieldError fe : e.getBindingResult().getFieldErrors()) {
          bindErrInfoList.add(new BindErrInfo(fe.getObjectName(), fe.getField(), fe.getRejectedValue(), fe.getDefaultMessage()));
      }

      res.putErrorData("BIND_ERROR_DETAILS", bindErrInfoList);
      return res;
  }
  
  // @Validated 에 의한 오류 처리
  @ExceptionHandler(ConstraintViolationException.class)

	// required 파라미터가 없는 경우 오류 처리
  @ExceptionHandler(MissingServletRequestParameterException.class)	

  // RequestParameter @Vaild 오류 처리
  @ExceptionHandler(MethodArgumentNotValidException.class)
} 

```

Parameter Validate

```java
@PostMapping("/latency")
public ApiRes latencyLog(@Valid @RequestBody LatencyReq logs) {
	... 
}

public class LatencyReq {
   @NotNull 
	 private String data;
} 

// null 입력시 MethodArgumentNotValidException 발생 

```