# 통합 테스트 시 db 초기화

실제 db 환경을 사용하는 통합 테스트를 수행할 때에는 각 테스트 케이스들 간의 독립성을 보장하기 위해 각 테스트 수행마다 db를 초기화해야 한다.  
각 테스트 수행시마다 실행하는 것은 afterEach로 등록하면 된다.  
db 초기화에서는 nestJS와 typeorm을 사용하는 환경에서 다음의 코드를 사용하면 된다.

```ts
let app: INestApplication;

beforeEach(async () => {
  const moduleFixture: TestingModule = await Test.createTestingModule({
    imports: [AppModule],
  }).compile();

  app = moduleFixture.createNestApplication();
  await app.init();
});

afterEach(async () => {
  const dataSource = app.get(DataSource);
  await dataSource.synchronize(true);
});
```

nestjs app에 등록되어 있는 DataSource를 받아온 다음, synchronize(true)를 해서 모든 데이터베이스 테이블을 drop 해주면 된다.  
(synchronize는 엔티티에 연결되어 있는 테이블에 대해서 생성해주는 메서드인데, 인자로 기존의 모든 테이블을 drop할지 여부를 받는다.)

참고자료: https://github.com/nestjs/nest/issues/409#issuecomment-390569628

위 참고자료에서 typeorm의 Connection은 deprecated되고 그 대신 DataSource가 사용되고 있으므로, 해당 부분만 변경했다.
