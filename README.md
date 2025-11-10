# Codi-It 개인 개발 리포트 

## 1. 📑프로젝트 개요 
- Codi-It은 패션 이커머스 플랫폼으로 구매자와 판매자를 위한 맞춤형 기능을 제공합니다. 구매자에게는 다양한 필터링과 정렬 기능을, 판매자에게는 상세한 대시보드를 지원하여 구매자와 판매자 모두에게 편리함을 선사합니다. 

---
## 2. ⚙️기술 스택
### -프레임워크
: Node.js (express) 
### -언어
: TypeScript
### -스키마/ORM
: Prisma
### -DATABASE
: PostgreSQL
### -API 문서화
: Swagger
### -테스트
: Jest, supertest
### -실시간 알림
: SSE(Server-Sent Events)
### -배포 
: AWS EC2, S3, Nginx, PM2, RDS
### -CI/CD
: Github Actions
### -코드 품질 관리
: ESLint + Prettier, Class-validator
### -협업 도구
: Discord, GitHub, Notion 

---
## 3. 담당 작업
### 1. Store API
: seller 계정으로 회원가입 시 store 생성 가능
- 스토어 등록
- 스토어 수정
- 스토어 상세 조회
- 내 스토어 상세 조회
- 내 스토어 등록 상품 조회
- 관심 스토어 등록
- 관심 스토어 해제
### 2. Notification API
: sse 단방향 통신으로 특정 이벤트 발생 시 사용자에게 실시간 알림 전송
- 실시간 알람
- 알람 조회(user type에 따른 알람조회)
- 알람 읽음 처리 
### 3. Dashboard API
: store의 전용 기능으로 판매 건수 및 금액 통계
- 대시보드 조회 
### 4. 그 외
- **시연영상 촬영**
- **unit test 작성** : store repository/service , notification repository/service , dashboard repository/service
- **metadata API 작성** : feat/metadata 브랜치 

---
## 4. 문제 해결 과정
### ProductTable에서 발생하는 'undefined.map' 런타임 에러를 해결하기 위해 백엔드 응답 구조를 수정
- 기존 코드
<img width="988" height="386" alt="getMyStoreProducts 기존 코드" src="https://github.com/user-attachments/assets/13c3c00e-6e20-4cea-b253-7c4f419c2ca1" />

    - 클라이언트(프론트엔드)는 객체가 아닌 **배열([])**을 받았고, 배열에는 list라는 속성이 없으므로 data?.list는 undefined가 되어 런타임 에러가 발생

- 변경 후 코드
<img width="1015" height="611" alt="getMyStoreProducts 변경된 코드" src="https://github.com/user-attachments/assets/7d2f07bc-2bbf-47e5-aff2-da6440621a1c" />
<img width="845" height="325" alt="getmystoreProduct-service" src="https://github.com/user-attachments/assets/4997404d-0146-47ab-aae1-b079ef79dd54" />



**[문제 원인]**
- Service layer에서 반환된 { list, totalCount } 객체 전체에 ProductResponseDto 변환을 적용하여, 최종 응답이 { list: [...], totalCount: 0 } 객체가 아닌 list 배열([...] ) 형태로 나갔음.
- 프론트엔드가 배열에서 data.list를 찾지 못하고 undefined가 되어 런타임 에러 발생.

**[해결]**
- Controller에서 DTO 변환을 list 배열에만 적용하고, 최종 응답 시 { list: DTO_LIST, totalCount: COUNT } 구조를 명시적으로 재구성하여 반환함.
- DB 조회 결과가 null/undefined일 경우를 대비하여 `productsWithStock || []` 방어 로직 추가.


  
### sse 연결 중 발생하는 에러를 해결하기 위해 인스턴스를 한번만 생성하도록 수정
- 기존 코드

```
// index.router.ts

const router = express.Router();

router.use('/users', userRouter);
router.use('/auth', authRouter);
router.use('/products', productRouter);
router.use('/inquiries', inquiryRouter);
router.use('/stores', StoresRouter(prisma));
router.use('/notifications', NotificationRouter(prisma));
router.use('/dashboard', DashboardRouter(prisma));

export default router;
```

- 변경 코드

```
// index.router.ts

const router = express.Router();

const sharedNotificationRepository = new NotificationRepository(prisma);
const sharedNotificationService = new NotificationService(sharedNotificationRepository);

router.use('/users', userRouter);
router.use('/auth', authRouter);
router.use('/products', productRouter(prisma, sharedNotificationService));
router.use('/inquiries', InquiryRouter(prisma, sharedNotificationService));
router.use('/stores', StoresRouter(prisma));
router.use('/notifications', NotificationRouter(prisma, sharedNotificationService));
router.use('/dashboard', DashboardRouter(prisma));
router.use('/cart', buildCartRouter());
router.use('/orders', orderRouter);
router.use('/metadata', metadataRouter);

```
**[문제 원인]** 
- InquiryRouter와 ProductRouter, NotificationRouter가 각각 NotificationService의 새 인스턴스를 생성하여, SSE 연결 클라이언트 목록(clients 맵)을 공유하지 못함.


**[해결]**
- Inquiry와 Product 기능을 Notification과 동일한 의존성 주입(DI) 패턴으로 통일 
- 메인 라우터 파일에서 NotificationRepository와 NotificationService의 인스턴스를 **sharedNotificationService**로 한 번만 생성.

**[추가 코드 변경]**
- 장시간 연결을 유지하기 위해 주기적으로 핑(Ping) 메시지 전송

```
const pingIntervalId = setInterval(() => {
      res.write(': keep-alive\n\n');
      if (res.flush) res.flush();
    }, PING_INTERVAL_MS);
```
---
## 5. 협업 및 피드백
- 코드리뷰 진행
  - 디스코드 웹훅 연결을 통해 리뷰어 지정 시 태그 추가되어 알람
  - 태그 된 후 하루 내로 코드리뷰 진행
  - 리뷰어의 approve 시 pr 올린 본인이 직접 merge 진행

- 작성 코드리뷰 목록
  1.	https://github.com/nb02-codi-it-team1/Codi-It-backend/pull/28
  2.	https://github.com/nb02-codi-it-team1/Codi-It-backend/pull/42
  3.	https://github.com/nb02-codi-it-team1/Codi-It-backend/pull/50
  4.	https://github.com/nb02-codi-it-team1/Codi-It-backend/pull/58
  5.	https://github.com/nb02-codi-it-team1/Codi-It-backend/pull/69
  등 ..

- 데일리스크럼 진행
  - issue에 데일리스크럼 생성된 후 각자 팀원이 오늘 할 일을 댓글로 작성

- 느낀점
  - 소통의 중요성
: 이번 고급 프로젝트에서는 데일리스크럼이 간략하게 진행되어 개인적으로 협업이 잘 진행되지 않은 것 같다는 아쉬움이 느껴짐.
데일리스크림을 마이크를 통해 진행하지 않고, 간단하게 오늘 할 일 정도만 issue 탭에 댓글로 작성하다보니, 팀원의 작업 진행 속도를 알기 어려웠고 서로가 느끼는 어려운 점 등을 공유하기가 어려웠음.
그러다보니 팀원들의 작업 속도가 제각각이고, 먼저 작업을 끝낸 사람도 선뜻 다른 사람을 도와주기 어려운 분위기가 형성됨.
또한 어려운 점에 대한 공유 없이 본인의 힘으로만 진행하려다 보니 결과적으로는 프로젝트를 기간 내에 완성하지 못하게 되었음.
이전 프로젝트에서는 데일리 스크럼을 진행하며 <어제 한 일, 오늘 할 일, 이 외의 논의 사항이나 어려운점> 등을 마이크를 통해 공유하면서도 '이게 무슨 도움이 될까?'라는 생각이 있었으나 이번 프로젝트를 진행하며
데일리 스크럼의 중요성을 알게 되었음. 간단한 내용의 회의이지만, 진행되고 되지 않고의 차이가 크고, 더욱 나아가 협업에서의 제일 중요한 규칙이라고 생각하게 됨.

  - 코드리뷰
: 이전 프로젝트에서는 코드리뷰를 진행하며, 내가 어디까지 코드를 꼼꼼하게 보아야하는지, 어느정도까지 코드 리뷰를 작성해야하는지 감도 잡히지 않았으나, 이번 프로젝트에서는 조금이나마 코드리뷰에 감이 조금 잡혔다는 생각이 들었음.
코드리뷰를 통해 팀원들의 코드 작성 스타일을 보면서 다양한 구현 방법을 보게 되었으며, 서로 아는 것을 공유하고, 프로젝트에 대한 더 나은 방향성을 논의할 수 있는 기회가 마련되는 것 같다고 느껴짐.

---
## 6. 코드 품질 및 최적화 
- ESLint 와 Prettier 설정을 통한 코드 스타일 통일 
- tsconfig 설정을 통한 까다로운 타입 설정
- class-validator 라이브러리 사용을 통한 dto 설정으로 타입 안정성 확보
- 의존성 주입 방식으로 클래스를 생성하여 높은 유영성과 테스트 용이성 확보
- 레이어드 아키텍쳐 설계를 통한 관심사 분리와 유지보수 용이성 확보

---
## 7. 향후 개선 사항 및 제안
- 추천 상품 로직 추가
  - 추천 상품을 리뷰가 많은 순, 혹은 구매가 많은 순으로 정렬하여 추천하는 기능
- 스토어 검색 기능
  - 현재는 상품 관련해서만 검색이 가능하고, 스토어 자세히보기를 위해서는 상품을 타고 들어가야지 가능한 상태임. 상점 또한 검색할 수 있도록 추가 기능을 만들면 좋을 거 같음.


