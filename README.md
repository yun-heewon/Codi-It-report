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
**시연영상 촬영**
**unit test 작성** : store repository/service , notification repository/service , dashboard repository/service
**metadata API 작성** : feat/metadata 브랜치 

---
## 4. 기능 구현 예시 

