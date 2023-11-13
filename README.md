## 보안

### CSRF (Cross-Site Request Forgery)

> 사이트 간 요청 위조
> 사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위 (수정, 삭제, 등록 등) 를 특정 웹사이트에 요청하게 하는 공격

<center>
    <img src='./public/csrf.jpg' width="75%">
</center>

#### 전제 조건

- 사용자가 서버로부터 인증을 받은 상태여야 한다.
- 쿠키 기반으로 서버 세션 정보를 획득할 수 있어야 한다.
- 공격자는 서버를 공격하기 위한 요청 방법에 대해 미리 파악하고 있어야 한다.

#### 공격 과정

1. 피해자가 서버에 로그인 후 세션 정보를 사용할 수 있는 **sessionId** 가 브라우저 쿠키에 저장된다.
2. 공격자는 피해자가 악성 스크립트 페이지를 방문하도록 유도한다.

- 악성 스크립트 페이지를 방문하도록 유도하는 방법 예시
  - 게시판이나 SNS 페이지에 악성 스크립트를 포스팅하여 클릭하도록 유도한다.
  - 메일로 악성 스크립트가 담긴 페이지 링크를 전달한다.

3. 피해자가 악성 스크립트가 담긴 페이지 접근시 쿠키에 저장된 **sessionId** 와 함께 서버에 요청된다.
4. 서버는 쿠키에 담긴 **sessionId** 를 통해 요청이 인증된 사용자로부터 온 것으로 판단하고 처리한다.

#### NodeJS 환경에서 방어 방법

1. CSRF 토큰 사용 (_서버에서 HTML 폼을 직접 생성하고 제공하는 경우_)
   - CSRF 토큰은 사용자 세션에 대한 요청이 신뢰할 수 있는지 확인하는데 사용
   - 서버에서 생성되어 클라이언트에 전달되며, 이후의 모든 요청에 이 토큰을 포함시켜 서버가 검증할 수 있게 한다.

```ts
import express from "express";
import session from "express-session";
import csrf from "csurf";
import { Request, Response } from "express";

const app = express();

// 세션 미들웨어 구성
app.use(
  session({
    /* 세션 설정 */
  })
);

// CSRF 미들웨어 구성
app.use(csrf());

app.get("/form", (req: Request, res: Response) => {
  // 폼의 hidden 타입 input 에 CSRF 토큰 포함
  res.send(`<form action="/process" method="POST">
              <input type="hidden" name="_csrf" value="${req.csrfToken()}">
              <input type="submit" value="Submit">
            </form>`);
});

app.post("/process", (req, res) => {
  // 요청 처리
});

app.listen(3000);
```

2. SameSite 쿠키 속성 설정

- 쿠키에 `SameSite` 속성을 설정하면 쿠키가 같은 사이트의 요청에서만 전송된다.

```ts
import express from "express";
import session from "express-session";

const app = express();

app.use(
  session({
    secret: "your secret key",
    cookie: {
      sameSite: "strict", // 'lax' 또는 'strict' 설정
    },
  })
);
```

3. Referrer 검증

- HTTP [Referrer 헤더](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Referer) 를 검증하여 요청이 신뢰할 수 있는 소스로부터 온 것인지 확인
- 같은 도메인 내의 페이지에 XSS 취약점이 있는 경우 CSRF 공격에 취약해질 수 있다.
- 좀 더 세밀하게 페이지 단위까지 일치하는지 검증하면 도메인 내의 타 페이지에서의 CSS 취약점에 의한 CSRF 공격을 방어할 수 있다.

#### 참고

- [CSRF 공격이란?](https://itstory.tk/entry/CSRF-%EA%B3%B5%EA%B2%A9%EC%9D%B4%EB%9E%80-%EA%B7%B8%EB%A6%AC%EA%B3%A0-CSRF-%EB%B0%A9%EC%96%B4-%EB%B0%A9%EB%B2%95)
- [SameSite](https://jjam89.tistory.com/231)
- [csurf](https://inpa.tistory.com/entry/NODE-%EB%B3%B4%EC%95%88-%F0%9F%93%9A-csurf-%EB%AA%A8%EB%93%88-%EC%82%AC%EC%9A%A9%EB%B2%95)
