---
title: "[spring MVC] Message Converter" 
date: 2021-07-15 19:29:24
tags:
	- spring
categories:
	- spring
	- spring MVC
	
---

# Message Converter 사용 이유
* @ResponseBody를 사용하면, Http body에 문자내용을 직접 반환하게 된다. 이때, viewResolver가 아니라 HttpMessageConverter가 동작하게 된다.
* 더 정확하게, spring MVC는 다음의 경우에 messageConverter를 적용한다.
  * Http 요청일때: `@ResquestBody`, `HttpEntity`
  * http 응답일때: `QresponseBody`, `HttpEntity`
___

# HttpMessageConverter Interface
```java
package org.springframework.http.converter;
...

public interface HttpMessageConverter<T> {

	boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);

	boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

	List<MediaType> getSupportedMediaTypes();

	default List<MediaType> getSupportedMediaTypes(Class<?> clazz) {
		return (canRead(clazz, null) || canWrite(clazz, null) ?
				getSupportedMediaTypes() : Collections.emptyList());
	}
	
	T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

	void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

}
```
* `canRead(), canWrite()`: converter가 해당 class, 미디어타입을 지원하는지 확인한다.
* `read() , write()` : 컨버터를 통해서 메시지를 읽고 쓴다.
___

# springBoot 기본 message converter

우선순위 | converter 
------- | ---------
 0 | ByteArrayHttpMessageConverter 
 1 | StringHttpMessageConverter
 2 | MappingJackson2HttpMessageConverter
* 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정한다. 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.

* converter별 지원 클래스타입과 미디어 타입

## Http request 데이터 읽기 흐름
1. http 요청이 오고, Controller에서 @RequestBody 또는 HttpEntity 파리미터를 사용한다고 가정하자.
2. messageConverter가 메시지를 읽을 수 있는지 확인하기 위해 canRead()를 호출한다. 이때, (1) 대상 class의 타입을 지원하는지 확인한다. e.g., byte[], String, Hellodata 등. (2) 요청의 미디어타입(content-type)을 지원하는지 확인한다. e.g., text/plain, application/json, */*
3. canRead() 조건을 만족하면 read()를 호출해서 객체를 생성하고 반환한다.
___

# messageConverter의 작동 시점
* 우리는 spring MVC 요청흐름을 파악한 바 있다. 
![이미지](/images/springMVC_structure.PNG)[^1]

  1. client가 Http 요청을 보내면, DispatcherServlet에서 HandlerMapping으로 handler를 조회한다. `@RequestMapping`의 경우, RequestMappingHandlerMapping가 실행한다. 
  2. handler를 찾은 후 HandlerAdapter로 handler를 실행할 수 있는 어댑터를 찾는다. @RequestMapping의  경우, **RequestMappingHandlerAdapter**가 실행되는데, 이 어댑터가 내부에서 해당 controller를 실행한다. 
  3. DispatcherServlet으로 modelAndView 반환한다.


* **`RequestMappingHandlerAdapter`에서 controller의 @RequestBody, HttpEntity, HttpServletRequest, Model 등을 처리해준다.**
![이미지](/images/ArgumentResolver요청흐름.PNG)[^2]

  1. RequestMappingHandlerAdapter는 **ArgumentResolver**를 호출해서 controller가 필요로 하는 parameter의 값(객체)를 생성한다.
  2. Adapter는 controller를 호출하면서 그 값을 넘겨준다.

___

## ArgumentResolver

* ArgumentResolver: `HandlerMethodArgumentResolver`Interface 
```java
package org.springframework.web.method.support;

public interface HandlerMethodArgumentResolver {
-
	boolean supportsParameter(MethodParameter parameter);

	@Nullable
	Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;

}
```
* ArgumentResolver 의 supportsParameter() 를 호출해서 해당 파라미터를 지원하는지 체크하고,
지원하면 resolveArgument() 를 호출해서 실제 객체를 생성한다. 그리고 이렇게 생성된 객체가 컨트롤러
호출시 넘어가는 것이다.

## ReturnValueHandler
* HandlerMethodReturnValuehandler는 응답 값을 변환하고 처리한다.
* ModelAndView, @ResponseBody, HttpEntity, String 등
https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-annreturn-types

____

## message Converter가 호출되는 곳
![이미지](/images/HTTPMessageConverter.PNG)
* 요청할 때: @ResponseBody는 HandlerMethodArgumentResolver를 구현한 RequestResponseBodyMethodProcessor에 의해 처리된다. (HttpEntity는 HttpEntityMethodProcessor)

![이미지](/images/ArgumentResolver_Diagram.PNG)

  1. supportsParameter에서 ResponseBody를 체크한 후,
  2. resolveArgument()에서 readWithMessageConverters를 사용한다.
  3. Object 만들어서 반환해준다.  

이 ArgumentResolver들이 messageConverter를 사용해서 필요한 객체를 생성한다.

```java
package org.springframework.web.servlet.mvc.method.annotation;

public class RequestResponseBodyMethodProcessor extends AbstractMessageConverterMethodProcessor {

	public RequestResponseBodyMethodProcessor(List<HttpMessageConverter<?>> converters) {
		super(converters);
	}

	public RequestResponseBodyMethodProcessor(List<HttpMessageConverter<?>> converters,
			@Nullable ContentNegotiationManager manager) {

		super(converters, manager);
	}

	public RequestResponseBodyMethodProcessor(List<HttpMessageConverter<?>> converters,
			@Nullable List<Object> requestResponseBodyAdvice) {

		super(converters, null, requestResponseBodyAdvice);
	}

	/**
	 * Complete constructor for resolving {@code @RequestBody} and handling
	 * {@code @ResponseBody}.
	 */
	public RequestResponseBodyMethodProcessor(List<HttpMessageConverter<?>> converters,
			@Nullable ContentNegotiationManager manager, @Nullable List<Object> requestResponseBodyAdvice) {

		super(converters, manager, requestResponseBodyAdvice);
	}


	@Override
	public boolean supportsParameter(MethodParameter parameter) {
		return parameter.hasParameterAnnotation(RequestBody.class);
	}

	@Override
	public boolean supportsReturnType(MethodParameter returnType) {
		return (AnnotatedElementUtils.hasAnnotation(returnType.getContainingClass(), ResponseBody.class) ||
				returnType.hasMethodAnnotation(ResponseBody.class));
	}

		@Override
	public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

		parameter = parameter.nestedIfOptional();
		Object arg = readWithMessageConverters(webRequest, parameter, parameter.getNestedGenericParameterType());
		String name = Conventions.getVariableNameForParameter(parameter);

		if (binderFactory != null) {
			WebDataBinder binder = binderFactory.createBinder(webRequest, arg, name);
			if (arg != null) {
				validateIfApplicable(binder, parameter);
				if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
					throw new MethodArgumentNotValidException(parameter, binder.getBindingResult());
				}
			}
			if (mavContainer != null) {
				mavContainer.addAttribute(BindingResult.MODEL_KEY_PREFIX + name, binder.getBindingResult());
			}
		}

		return adaptArgumentIfNecessary(arg, parameter);
	}

	@Override
	protected <T> Object readWithMessageConverters(NativeWebRequest webRequest, MethodParameter parameter,
			Type paramType) throws IOException, HttpMediaTypeNotSupportedException, HttpMessageNotReadableException {

		HttpServletRequest servletRequest = webRequest.getNativeRequest(HttpServletRequest.class);
		Assert.state(servletRequest != null, "No HttpServletRequest");
		ServletServerHttpRequest inputMessage = new ServletServerHttpRequest(servletRequest);

		Object arg = readWithMessageConverters(inputMessage, parameter, paramType);
		if (arg == null && checkRequired(parameter)) {
			throw new HttpMessageNotReadableException("Required request body is missing: " +
					parameter.getExecutable().toGenericString(), inputMessage);
		}
		return arg;
	}
```
* 응답 시: @ResponseBody, HttpEntity를 처리하는 ReturnValueHandler가 messageConverter를 호출해서 응답 결과를 만든다.

# Reference
* 김영한(2021-03-21), Spring MVC 1편, Ch6 S13 ~ S14