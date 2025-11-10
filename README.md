# Zaranara - 경제 교육 플랫폼

Zaranara는 경제 뉴스 요약, 금융 퀴즈, 예산 관리 등을 제공하는 종합 경제 교육 플랫폼입니다.

## 프로젝트 구조

```
zaranara/
├── Zaranara_FE/        # Vue 3 + TypeScript Frontend
├── Zaranara_BE/        # Spring Boot Backend
├── Zaranara_AI/        # FastAPI AI Server (뉴스 요약, 분석)
├── docker-compose.yml  # Docker Compose 설정
└── .env                # 환경 변수 설정
```

## 주요 기능

### 1. AI 뉴스 요약
- 네이버 뉴스 API를 활용한 경제 뉴스 수집
- OpenAI GPT를 활용한 뉴스 요약 및 쉬운 설명
- 경제 용어 설명 자동 생성

### 2. 금융 퀴즈
- 난이도별 금융 지식 퀴즈
- 사용자 정답률 추적

### 3. 예산 관리
- 월별 소비 내역 기록
- AI 기반 소비 분석 리포트
- 예산 대비 실제 지출 비교

### 4. 소셜 로그인
- Google OAuth2
- Naver OAuth2
- Kakao OAuth2

## 기술 스택

### Frontend (Zaranara_FE)
- Vue 3 (Composition API)
- TypeScript
- Vue Router
- Axios
- Vite

### Backend (Zaranara_BE)
- Spring Boot 3.4
- Java 17
- Spring Security + JWT
- Spring Data JPA
- MariaDB
- OAuth2 Client

### AI Server (Zaranara_AI)
- FastAPI
- OpenAI GPT API
- Python 3.11+
- Pydantic

### Infrastructure
- Docker & Docker Compose
- Nginx (Frontend 프록시)
- MariaDB 11

## 빠른 시작

### 1. 환경 변수 설정

루트 디렉토리에 `.env` 파일을 생성하고 다음 값들을 설정하세요:

```bash
# Database
DB_NAME=zara
DB_PASSWORD=your_secure_password

# JWT
JWT_SECRET=your-256-bit-secret-key-at-least-32-characters-long
JWT_EXPIRATION=86400000

# OAuth2
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
NAVER_CLIENT_ID=your-naver-client-id
NAVER_CLIENT_SECRET=your-naver-client-secret
KAKAO_CLIENT_ID=your-kakao-client-id
KAKAO_CLIENT_SECRET=your-kakao-client-secret

# OpenAI
OPENAI_API_KEY=your-openai-api-key
OPENAI_MODEL=gpt-4o-mini

# Application
APP_REDIRECT_URL=http://localhost:5173/auth/callback
SPRING_PROFILES_ACTIVE=prod
```

`.env.example` 파일을 참고하세요.

### 2. Docker Compose로 전체 스택 실행

```bash
# 모든 서비스 빌드 및 시작
docker-compose up --build

# 백그라운드 실행
docker-compose up -d --build
```

### 3. 서비스 접속

- **Frontend**: http://localhost:5173
- **Backend API**: http://localhost:8080
- **AI Server**: http://localhost:8000
- **Swagger Docs**: http://localhost:8080/swagger-ui.html
- **FastAPI Docs**: http://localhost:8000/docs

### 4. 개발 환경 실행

각 서비스를 개별적으로 실행할 수 있습니다:

#### Frontend
```bash
cd Zaranara_FE
npm install
npm run dev
```

#### Backend
```bash
cd Zaranara_BE
./gradlew bootRun
```

#### AI Server
```bash
cd Zaranara_AI
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.api:app --reload --host 0.0.0.0 --port 8000
```

#### Database
```bash
# MariaDB 직접 실행 또는 Docker 사용
docker run -d \
  --name zaranara-db \
  -e MYSQL_ROOT_PASSWORD=SqlDba-1 \
  -e MYSQL_DATABASE=zara \
  -p 3379:3306 \
  mariadb:11
```

## API 문서

### Backend API
- Swagger UI: http://localhost:8080/swagger-ui.html
- 주요 엔드포인트:
  - `POST /api/auth/login` - 로그인
  - `GET /api/quiz/random` - 랜덤 퀴즈
  - `GET /api/budget/{userId}` - 사용자 예산 조회
  - `POST /api/budget/{userId}` - 예산 생성

### AI Server API
- FastAPI Docs: http://localhost:8000/docs
- 주요 엔드포인트:
  - `POST /v1/pipeline/run` - 뉴스 요약 파이프라인
  - `POST /v1/report/monthly` - 월간 소비 분석 리포트
  - `GET /health` - 헬스체크

## 프로젝트 상세 문서

- [Docker 빠른 시작 가이드](DOCKER-QUICK-START.md)
- [AI Server README](Zaranara_AI/README.md)
- [Frontend Integration Guide](Zaranara_FE/API_INTEGRATION_GUIDE.md)
- [Budget API Guide](Zaranara_FE/BUDGET_API_GUIDE.md)

## 데이터베이스 스키마

주요 테이블:
- `users` - 사용자 정보
- `quiz` - 퀴즈 문제
- `user_quiz_history` - 퀴즈 정답 기록
- `budgets` - 예산 계획
- `expenses` - 지출 내역
- `news_history` - 뉴스 요약 기록

상세 스키마는 `Zaranara_BE/sql/` 디렉토리를 참조하세요.

## 트러블슈팅

### 포트 충돌
다른 서비스가 포트를 사용 중인 경우 `docker-compose.yml`에서 포트를 변경하세요.

### 빌드 실패
```bash
# 캐시 없이 재빌드
docker-compose build --no-cache
docker-compose up -d
```

### 데이터베이스 초기화
```bash
# 볼륨 삭제 후 재시작
docker-compose down -v
docker-compose up -d
```

### 로그 확인
```bash
# 모든 서비스 로그
docker-compose logs -f

# 특정 서비스 로그
docker-compose logs -f backend
docker-compose logs -f ai
docker-compose logs -f frontend
```

## 보안 주의사항

1. `.env` 파일을 Git에 커밋하지 마세요
2. 프로덕션 환경에서는 강력한 JWT 시크릿 키를 사용하세요
3. OAuth2 클라이언트 시크릿을 안전하게 관리하세요
4. 데이터베이스 비밀번호를 기본값에서 변경하세요

## 라이선스

이 프로젝트는 교육용 프로젝트입니다.

## 기여

이슈와 풀 리퀘스트를 환영합니다.
