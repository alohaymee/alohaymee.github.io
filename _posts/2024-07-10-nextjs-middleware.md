---
layout: post
title: "Next.js Middleware 정리"
date: 2024-07-10 13:16:23 +0900
categories: Nextjs
---

스터디 겸 [Nextjs Middleware][origin] 를 정리해보았다.

## Middleware
### 요약
미들웨어는 요청을 받아 응답을 리턴하는 중간에서 rewriting, redirecting, 요청 데이터 수정, 응답 헤더 수정 등 여러가지 처리를 할 수 있다.  

미들웨어는 캐시된 컨텐츠와 라우팅 전 단계에서 실행된다.

### 이런 경우 미들웨어를 사용하자
미들웨어를 이용하면 성능, 보안, 사용자 경험에 상당한 개선을 가져올 수 있다. 특히 아래의 경우 미들웨어의 효과를 볼 수 있다.  
1. 인증 및 권한 부여  
    특정 페이지나 API 경로에 대해 접근을 허용하기 전, 사용자의 신원을 확인하고 세션 쿠키를 검사할 수 있다.
2. 서버단 리디렉션  
    특정 조건(접속 지역, 사용자 역할 등)으로 사용자를 리디렉션 시킬 수 있다.
3. 경로 재작성  
    요청 속성을 기반으로 API 또는 페이지 요청에 대한 경로를 동적으로 재작성하여 A/B 테스트, 롤아웃 기능, 레거시 경로 등을 지원한다.
4. 봇 탐지  
    봇 트래픽을 탐지하고 차단하여 리소스를 보호한다.
5. 로깅 및 분석  
    페이지나 API에 의한 처리가 이루어지기 전에 요청 데이터를 캡처하고 분석한다.
6. 기능 플래깅  
    기능 출시 또는 테스트를 위해 특정 기능들을 동적으로 활성화하거나 비활성화 한다.

[origin]: [https://nextjs.org/docs/app/building-your-application/routing/middleware#use-cases]

### 다만 이런 경우라면 주의하자
미들웨어가 최적의 접근 방식이 아닌 경우에 대해 인지하고 있는 것도 중요하다.

1. 복잡한 데이터 가져오기 및 조작  
    미들웨어는 직접 데이터를 가져오거나 조작하기 위해 설계된 것이 아니다. 이러한 작업은 라우트 핸들러나 서버 레벨의 유틸리티에 의해 수행되어야 한다.

2. 무거운 계산 작업  
    미들웨어는 가벼워야하고 빠르게 응답할 수 있어야 한다. 그렇지 않으면 페이지 로딩에 지연이 발생할 수 있다. 무거운 계산 작업이나 오래 걸리는 처리 작업은 라우트 핸들러가 위임해야 한다.

3. 광범위한 세션 관리  
    미들웨어에서 기본적인 세션 관리는 가능하지만, 광범위한 세션 관리는 별도의 인증 서비스나 라우트 핸들러에서 처리되어야 한다.

4. 직접적인 데이터베이스 작업
    미들웨어에서 직접적인 데이터베이스 작업을 수행하는 것은 권장되지 않는다. 데이터베이스와 상호작용은 라우트 핸들러나 서버 측 유틸리티에서 처리되어야 한다.

### 미들웨어 사용하기
프로젝트 루트에 `middleware.ts` (또는 `.js`) 파일을 생성한다.(App 라우터 방식인 경우 `app` 폴더와 동일한 위치, Pages 라우터 방식인 경우 `pages` 폴더와 동일한 위치에 생성한다.)

프로젝트 당 하나의 `middleware.ts` 파일만 지원된다. 모듈화가 필요한 경우 파일 단위로 분리하고 `middleware.ts` 파일에서 `import` 할 수 있다. 이 방식으로 경로별 미들웨어를 깔끔하게 관리할 수 있다. 단일 `middleware.ts` 파일을 적용하여 구성을 단순화 함으로서 잠재적인 충돌을 예방할 수 있다. 또한 여러 계층의 미들웨어가 생성되는 것을 막을 수 있어 성능을 최적화 할 수 있다.

### 미들웨어를 적용할 경로 설정하기 
미들웨어는 모든 라우트에 적용된다. 미들웨어가 주어지면, 타겟을 정확히 하기 위해 matchers 를 사용하는것이 중요하다. 실행 순서는 다음과 같다.

`headers` from `next.config.js`  
`redirects` from `next.config.js`  
Middleware (`rewrites`, `redirects`, etc.)  
`beforeFiles` (`rewrites`) from `next.config.js`  
Filesystem routes (`public/`, `_next/static/`, `pages/`, `app/`, etc.)  
`afterFiles` (rewrites) from `next.config.js`  
Dynamic Routes (`/blog/[slug]`)  
`fallback` (`rewrites`) from `next.config.js`  

미들웨어를 실행할 타겟을 정하는 정하는 2가지 방법이 있다.

1. [matcher 설정](#matcher)
2. [조건문 사용](#조건문-사용)


#### Matcher
matcher 를 설정하면 필터링되어 설정된 경로에서만 미들웨어가 실행된다.  

matcher 설정 예시 1. 특정 경로
```javascript
export const config = {
  matcher: "/v1/enter/*", // /v1/enter 경로 포함, 하위 경로 모두 해당됨
};
```
matcher 설정 예시 2. 배열
```javascript
export const config = {
  matcher: ["/v1/enter/*", "/v1/instructor/attendance"],
};
```
matcher 설정 예시 3. 정규식
```javascript
export const config = {
  matcher: [
    /*
     * 다음으로 시작하는 경로는 제외한다.
     * - api (API routes)
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     * - v1/login
     */
    "/((?!api|_next/static|_next/image|favicon.ico|v1/login).*)",
  ],
};
```
matcher 설정 예시 4. `missing`, `has` 배열 사용하여 미들웨어 통과 시키기  
```javascript
export const config = {
  matcher: [
    {
      source: "/((?!v1/login|_next/static|_next/image|favicon.ico).*)",
      has: [{ type: "header", key: "x-present" }],
      missing: [{ type: "header", key: "x-missing", value: "prefetch" }],
    },
  ],
};
```
- `missing`, `has` 둘 중 하나만 사용하거나 둘 다 사용할 수 있음  

> `matcher` 는 빌드 시 분석될 수 있는 상수만 설정이 가능하다. 

matcher 설정 규칙  
1. `/` 로 시작해야만 한다.
2. 네임드 파라미터를 지원한다. `/about/:path` 로 설정했다면 `/about/a`, `/about/b` 모두 해당되나 `/about/a/b` 는 해당되지 않는다.
3. 네임드 파라미터에 수정자를 허용한다. `/about/:path*` 는 `/about/a/b/c` 와 일치한다. 
    - `*` 는 0개 이상을 의미한다.
    - `?` 는 0개 또는 1개를 의미한다.
    - `+` 는 1개 이상을 의미한다.
4. 괄호 안에 정규 표현식을 사용할 수 있다. `/abount/(.*)` 는 `/about/:path*` 와 일치한다. 

#### 조건문 사용
matcher 는 단순한 필터링만 필요한 경우, 복잡한 처리가 필요하다면 조건문을 피할 수 없다.  
예시
```javascript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
 
export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith('/about')) {
    return NextResponse.rewrite(new URL('/about-2', request.url))
  }
 
  if (request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.rewrite(new URL('/dashboard/user', request.url))
  }
}
```

### NextResponse
`NextResponse` 로 이러한 작업을 할 수 있다.
- 들어온 요청을 다른 URL 로 `redirect`
- 주어진 URL 로 응답을 `rewrite`
- API 경로, `getServerSideProps` 에서 요청 헤더 설정 및 `rewrite`
- 응답 쿠키 설정
- 응답 헤더 설정  

응답을 생성하기 위해서 `rewrite` 로 페이지나 다른 경로로 보내거나 `NextResponse` 에서 바로 응답을 생성할 수 있다.

### Using Cookie
일반적으로 쿠키는 요청 헤더의 `Cookie` 에, 응답 헤더의 `Set-Cookie` 에 담긴다. Next.js 에서는 쿠키에 대한 접근 편의성을 위해 NextRequest, NextResponse 에서 
`cookies` 확장을 제공한다.

1. 요청에서 사용할 수 있는 메소드: `get`, `getAll`, `set`, `delete`, `has`(쿠키 여부 체크), `clear`(모든 쿠키 삭제)
2. 응답에서 사용할 수 있는 메소드: `get`, `getAll`, `set`, `delete`

예시
```javascript
export function middleware(request: NextRequest) {
  // 쿠키 가져오기
  let refreshToken = request.cookies.get("refresh-token");

  // 모든 쿠키 가져오기
  const allCookies = request.cookies.getAll();

  // 쿠키 체크하기
  request.cookies.has("refresh-token");

  // 쿠키 삭제하기
  request.cookies.delete("refresh-token");

  // 응답에 쿠키 설정하기
  const response = NextResponse.next();
  response.cookies.set("refresh-token", refreshToken);

  return response;
}
```

### Headers 설정하기
NextRequest, NextResponse 를 이용하여 요청과 응답 헤더를 설정할 수 있다.
```javascript
export function middleware(request: NextRequest) {
  const newRequestHeaders = new Headers();

  // request header 설정
  const response = NextResponse.next({
    request: {
      headers: newRequestHeaders,
    },
  });

  // response header 추가
  response.headers.set("set-response-header", "hello");
  return response;
}
```

### 미들웨어에서 바로 응답하기
`Response`, `NextResponse` 를 이용하여 미들웨어에서 바로 응답을 보낼 수 있다.
예시
```javascript
export function middleware(request: NextRequest) {
  if (!isAuthenticated(request)) {
    return Response.json(
      { success: false, message: 'authentication failed' },
      { status: 401 }
    )
  }
}
```

### 이 외에도 미들웨어에서 가능한 작업들
- CORS 설정으로 특정 도메인 허용이 가능하다.
- `NextFetchEvent` 의 `waitUntil` 메소드를 이용해서 백그라운드 작업 수행이 가능하다.

### 끝
잘 숙지하고 있으면 유용하게 쓸데가 많을 것 같다. 스프링 부트에서는 spring security, AOP 등에서 고루 적용해야할 것들을 Next.js 에서는 미들웨어에서 다 하는구나. Apache, Nginx 등에서 설정했던 것들도 여기서 한번에 다 설정이 가능하구나. 좋네..