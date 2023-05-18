# e2e 테스트 시 req.cookies가 없을 때

## 1. 에러 상황

supertest를 이용하여 e2e 테스트를 작성하는 과정에서 req.cookies 값이 계속 undefined가 나오는 문제가 있었다.  
supertest에서 쿠키를 set하는 코드는 아래와 같이 작성한다.

```ts
await request(app.getHttpServer())
  .get("/api/post")
  .set("Cookie", ["abcd=1234"])
  .expect(HttpStatus.OK);
```

위와 같이 작성했는데도, server 측에서 req.cookies로 쿠키에 접근하면 계속 undefined가 나왔다.

## 에러 원인

위 현상은 cookie-parser가 제대로 동작하지 못하고 있기 때문에 발생한다.
cookie-parser는 header 안에 있는 Cookie 필드의 값을 파싱해서 객체의 형태로 req.cookies에 넣어준다.

코드를 확인해보니, 테스트 모듈에서 받은 Nest APP을 실행할 때 cookie-parser 미들웨어를 넣어주지 않아서 발생한 에러였다.  
테스팅 모듈을 구성하는 코드는 다음과 같이 되어 있었다.

```ts
const moduleFixture: TestingModule = await Test.createTestingModule({
  imports: [AppModule],
}).compile();

app = moduleFixture.createNestApplication();
await app.init();
```

위 코드의 어디에도 cookieParser 미들웨어를 등록해주는 부분이 존재하지 않는다.

## 3. 해결 방법

app.init()을 하기 전에 미들웨어를 등록해주면 문제가 해결된다.

```ts
import * as cookieParser from "cookie-parser";

const moduleFixture: TestingModule = await Test.createTestingModule({
  imports: [AppModule],
}).compile();

app.use(cookieParser());
app = moduleFixture.createNestApplication();
await app.init();
```
