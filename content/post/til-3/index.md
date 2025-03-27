---
title: Spring Filter 에서 예외 발생시 RestControllerAdvice 로 처리 안됨
description:
slug: til-3
date: 2025-03-27 00:00:00+0000
image:
weight: 1
categories:
  - til
tags:
  - 스프링
  - JPA
  - 트러블 슈팅
---

## 개요
JPA를 사용한 일정 관리 앱을 개발하던 중에 발생한 문제이다.

필터를 통해 세션에 정보가 있는지 여부로 로그인 여부를 판별하고 있다.

## 트러블 슈팅

### 배경
필터를 통해 세션에 정보가 있는지 여부로 로그인 여부를 판별하고 있다.

로그인이 안된 상태로 로그인 필수 path 에 접근시 예외를 던지게 해두었다. 

그리고 해당 예외는 RestControllerAdvice 를 통해 예외를 처리하도록 해두었다.

```java
public class LoginFilter implements Filter {

    private static final String[] PUBLIC_PATH_LIST = {"/users/register", "/users/login"};

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain filterChain
    ) throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;

        String requestURI = req.getRequestURI();

        boolean isPrivatePath = !PatternMatchUtils.simpleMatch(PUBLIC_PATH_LIST, requestURI);
        if (isPrivatePath) {
            HttpSession session = req.getSession();

            if (session == null || session.getAttribute("userId") == null) {
                throw new RuntimeException("Please login first");
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

### 발단

개요와 같이 적용하고 실제로 로그인이 안된 상태에서 로그인 필수 path 에 접근하면 공통 예외 처리가 동작할 줄 알았다.
하지만 RestControllerAdvice 를 통해 공통으로 예외를 처리해둔 곳이 기능하지 않는다.

### 전개

이런 문제가 발생하는 이유는 RestControllerAdvice 는 컨트롤러에서 발생한 예외를 잡을 순 있지만, 
컨트롤러에 도달하기 전 필터에서 발생한 예외는 잡을 수 없기 때문이다.

이를 해결하기 위해 예외를 던지는 것이 아닌 응답에 예외를 바로 담도록 수정하였다.

### 결말

```java
public class LoginFilter implements Filter {

    private static final String[] PUBLIC_PATH_LIST = {"/users/register", "/users/login"};

    @Override
    public void doFilter(
            ServletRequest request,
            ServletResponse response,
            FilterChain filterChain
    ) throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;

        String requestURI = req.getRequestURI();

        boolean isPrivatePath = !PatternMatchUtils.simpleMatch(PUBLIC_PATH_LIST, requestURI);
        if (isPrivatePath) {
            HttpSession session = req.getSession();

            if (session == null || session.getAttribute("userId") == null) {
                exceptionHandler(requestURI, res);
                return;
            }
        }

        filterChain.doFilter(request, response);
    }

    private void exceptionHandler(String requestURI, HttpServletResponse res) {
        res.setStatus(HttpStatus.UNAUTHORIZED.value());
        res.setContentType("application/json");
        res.setCharacterEncoding("UTF-8");
        try {
            ObjectMapper objectMapper = new ObjectMapper();
            objectMapper.registerModule(new JavaTimeModule());

            String json = objectMapper.writeValueAsString(
                    ErrorResponse.of(
                            HttpStatus.UNAUTHORIZED,
                            "Please log in and try again.",
                            requestURI)
            );
            res.getWriter().write(json);
        } catch (Exception e) {
            log.error("LoginFilter - exceptionHandler {}", e.getMessage());
        }
    }
}
```

최종적으로 위와 같은 코드를 작성하였다.

로그인이 안된 상태에서 로그인 필수 path 에 접근해보면 정상적으로 공통 오류 메시지가 응답되는 것을 확인할 수 있다.

## 마무리

이번 트러블 슈팅을 요약하면 다음과 같습니다.

1. 웹 애플리케이션을 작업하다 보면 필터에서 예외를 처리해야하는 경우가 생깁니다.
2. 스프링 시큐리티를 사용할 경우 처리할 수 있는 방법이 따로 있습니다. 하지만 이번 경우에는 스프링 시큐리티를 사용하지 않기 때문에 다른 방법으로 해결해야 합니다.
3. 이번 경우에는 필터에서 예외를 던지는 것이 아닌 응답에 바로 예외 공통 메시지를 넣음으로써 해결이 되었습니다.
