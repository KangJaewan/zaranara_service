# Zaranara Docker Quick Start Guide

이 가이드는 Docker Compose를 사용하여 Zaranara 전체 스택(Frontend, Backend, AI Server, Database)을 한 번에 실행하는 방법을 설명합니다.

## 📋 사전 요구사항

- Docker Desktop 설치 (https://www.docker.com/products/docker-desktop)
- Docker Compose 설치 (Docker Desktop에 포함됨)
- 최소 8GB RAM, 20GB 디스크 공간

## 🚀 빠른 시작

### 1. 환경 변수 설정

루트 디렉토리의 `.env` 파일이 올바르게 설정되어 있는지 확인하세요:

```bash
# .env 파일 확인
cat .env
```

필수 환경 변수:
- `DB_PASSWORD`: MariaDB 비밀번호
- `JWT_SECRET`: JWT 토큰 시크릿 키
- `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`: Google OAuth2 인증 정보
- `NAVER_CLIENT_ID`, `NAVER_CLIENT_SECRET`: Naver OAuth2 인증 정보
- `KAKAO_CLIENT_ID`, `KAKAO_CLIENT_SECRET`: Kakao OAuth2 인증 정보
- `OPENAI_API_KEY`: OpenAI API 키

### 2. Docker Compose로 전체 스택 실행

```bash
# 모든 서비스 빌드 및 시작
docker-compose up --build

# 백그라운드 실행
docker-compose up -d --build
```

### 3. 서비스 접속

서비스가 모두 시작되면 다음 URL로 접속할 수 있습니다:

- **Frontend (Vue 3)**: http://localhost:5173
- **Backend (Spring Boot)**: http://localhost:8080
- **AI Server (FastAPI)**: http://localhost:8000
- **Swagger API 문서**: http://localhost:8080/docs
- **MariaDB**: localhost:3379 (컨테이너 내부: 3306)

### 4. 로그 확인

```bash
# 모든 서비스 로그 보기
docker-compose logs -f

# 특정 서비스 로그만 보기
docker-compose logs -f frontend
docker-compose logs -f backend
docker-compose logs -f ai
docker-compose logs -f db
```

### 5. 서비스 중지 및 재시작

```bash
# 서비스 중지 (컨테이너 유지)
docker-compose stop

# 서비스 재시작
docker-compose start

# 서비스 중지 및 컨테이너 삭제
docker-compose down

# 볼륨까지 삭제 (데이터베이스 데이터도 삭제됨)
docker-compose down -v
```

## 🏗️ 서비스 구성

### 1. MariaDB (Database)
- **컨테이너명**: `zaranara-db`
- **포트**: 3379:3306
- **볼륨**: `mariadb_data` (영구 저장)
- **초기 데이터**: `Zaranara_BE/sql/` 디렉토리의 SQL 파일 자동 실행

### 2. AI Server (FastAPI)
- **컨테이너명**: `zaranara-ai`
- **포트**: 8000:8000
- **기능**: 뉴스 수집/요약, 월별 소비 분석 AI
- **Health Check**: `/health` 엔드포인트

### 3. Backend (Spring Boot)
- **컨테이너명**: `zaranara-backend`
- **포트**: 8080:8080
- **프로필**: `prod` (환경변수로 변경 가능)
- **의존성**: MariaDB, AI Server
- **Health Check**: `/actuator/health` 엔드포인트

### 4. Frontend (Vue 3 + Nginx)
- **컨테이너명**: `zaranara-frontend`
- **포트**: 5173:80
- **빌드**: Multi-stage build (Node.js → Nginx)
- **의존성**: Backend

## 🔧 트러블슈팅

### 포트 충돌 문제

다른 서비스가 이미 포트를 사용 중인 경우:

```bash
# 포트 사용 확인
lsof -i :5173  # Frontend
lsof -i :8080  # Backend
lsof -i :8000  # AI Server
lsof -i :3379  # Database

# docker-compose.yml에서 포트 변경
# 예: "5173:80" → "다른포트:80"
```

### 빌드 실패

```bash
# 캐시 없이 재빌드
docker-compose build --no-cache

# 특정 서비스만 재빌드
docker-compose build --no-cache backend
```

### 데이터베이스 초기화

```bash
# 볼륨 삭제 후 재시작
docker-compose down -v
docker-compose up -d
```

### 컨테이너 상태 확인

```bash
# 실행 중인 컨테이너 확인
docker-compose ps

# 컨테이너 상세 정보
docker inspect zaranara-backend

# Health check 상태 확인
docker ps --format "table {{.Names}}\t{{.Status}}"
```

### 네트워크 문제

```bash
# Docker 네트워크 확인
docker network ls
docker network inspect web-mini_zaranara-network

# 네트워크 재생성
docker-compose down
docker network prune
docker-compose up -d
```

## 📊 개발 vs 프로덕션

### 개발 환경 (로컬)

```bash
# 각 서비스를 개별적으로 실행
cd Zaranara_FE && npm run dev
cd Zaranara_BE && ./gradlew bootRun
cd Zaranara_AI && uvicorn app.api:app --reload
```

### 프로덕션 환경 (Docker)

```bash
# .env 파일에서 프로필 변경
SPRING_PROFILES_ACTIVE=prod

# Docker Compose로 실행
docker-compose up -d --build
```

## 🔒 보안 주의사항

1. **환경 변수 보호**: `.env` 파일을 Git에 커밋하지 마세요
2. **시크릿 키 변경**: 프로덕션 환경에서는 반드시 강력한 시크릿 키 사용
3. **OAuth2 리다이렉트 URL**: 프로덕션 도메인에 맞게 수정
4. **데이터베이스 비밀번호**: 기본값 변경 권장

## 🧪 테스트

```bash
# Health check 테스트
curl http://localhost:8000/health    # AI Server
curl http://localhost:8080/actuator/health    # Backend
curl http://localhost:5173    # Frontend

# API 테스트
curl http://localhost:8080/api/quiz/random?difficulty=하
```

## 📝 유용한 명령어

```bash
# 컨테이너 셸 접속
docker exec -it zaranara-backend /bin/sh
docker exec -it zaranara-db mariadb -u root -p

# 리소스 사용량 확인
docker stats

# 디스크 사용량 확인
docker system df

# 사용하지 않는 리소스 정리
docker system prune -a
```

## 🆘 도움말

문제가 발생하면:

1. 로그 확인: `docker-compose logs -f [서비스명]`
2. Health check 상태 확인: `docker-compose ps`
3. 컨테이너 재시작: `docker-compose restart [서비스명]`
4. 완전히 재빌드: `docker-compose down && docker-compose up --build`

더 자세한 내용은 `README-DOCKER.md`를 참고하세요.
