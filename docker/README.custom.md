# Dify Custom Docker Setup

이 디렉토리는 커스터마이즈된 Dify Docker 설정을 포함합니다.

## 🚀 빠른 시작

### 1. 환경 변수 설정

```bash
# .env.example을 복사하여 .env 생성
cp .env.example .env

# SECRET_KEY 생성 (필수)
openssl rand -base64 42
# 생성된 키를 .env의 SECRET_KEY에 입력

# 포트 설정 (기본값: 8080)
EXPOSE_NGINX_PORT=8080
EXPOSE_NGINX_SSL_PORT=8443
```

### 2. 서비스 시작

```bash
# Weaviate 프로필로 시작
docker compose --profile weaviate up -d

# 모든 컨테이너 상태 확인
docker compose ps
```

### 3. 웹 인터페이스 접속

- URL: http://localhost:8080
- 초기 관리자 계정 생성 필요

## 📦 구성 요소

### 서비스 목록

| 서비스 | 포트 | 설명 |
|--------|------|------|
| nginx | 8080, 8443 | 리버스 프록시 |
| api | 5001 (내부) | Dify API 서버 |
| worker | - | Celery Worker |
| worker_beat | - | Celery Beat |
| web | 3000 (내부) | 프론트엔드 |
| db | 5432 (내부) | PostgreSQL 15 |
| redis | 6379 (내부) | Redis 6 |
| weaviate | 8080 (내부) | 벡터 데이터베이스 |
| sandbox | - | 코드 실행 샌드박스 |
| plugin_daemon | 5003 | 플러그인 관리 |

### 벡터 데이터베이스

**기본값**: Weaviate 1.19.0

다른 벡터 DB로 변경:
```bash
# .env 파일에서
VECTOR_STORE=qdrant    # Qdrant 사용
VECTOR_STORE=pgvector  # pgvector 사용
VECTOR_STORE=milvus    # Milvus 사용
```

## 🔒 보안 설정

### 필수 변경 사항

프로덕션 환경에서는 다음 값들을 반드시 변경하세요:

```bash
# .env 파일
SECRET_KEY=[openssl rand -base64 42로 생성]
DB_PASSWORD=[강력한 비밀번호]
REDIS_PASSWORD=[강력한 비밀번호]
WEAVIATE_API_KEY=[랜덤 생성]
```

### .env 파일 보안

- `.env` 파일은 절대 Git에 커밋하지 마세요
- `.gitignore`에 포함되어 있습니다
- `.env.example`만 버전 관리됩니다

## 💾 데이터 영속성

### 볼륨 위치

```
./volumes/
├── weaviate/      # Weaviate 데이터
├── db/data/       # PostgreSQL 데이터
├── redis/data/    # Redis 스냅샷
├── app/storage/   # 업로드된 파일
└── sandbox/       # Sandbox 설정
```

### 백업

```bash
# 전체 백업
tar -czf dify-backup-$(date +%Y%m%d).tar.gz volumes/

# 데이터베이스만 백업
docker exec docker-db-1 pg_dump -U postgres dify > backup.sql
```

## 🔧 문제 해결

### 컨테이너 재시작

```bash
docker compose restart
```

### 로그 확인

```bash
# 모든 로그
docker compose logs

# 특정 서비스
docker compose logs api
docker compose logs weaviate
```

### 디스크 공간 확보

```bash
# Docker 정리
docker system prune -a --volumes -f
```

### Weaviate READ-ONLY 해제

디스크 공간 확보 후:
```bash
docker compose restart weaviate
```

## 📊 시스템 요구사항

- **최소 메모리**: 4GB RAM
- **권장 메모리**: 8GB RAM
- **최소 디스크**: 20GB 여유 공간
- **필수 포트**: 8080, 8443 (변경 가능)

## 🌐 네트워크

### 네트워크 구성

- `docker_default` (172.18.0.0/16): 일반 서비스 통신
- `docker_ssrf_proxy_network` (172.19.0.0/16): SSRF 보호용 격리 네트워크

### DNS 해상도

모든 서비스는 서비스명으로 접근 가능:
- `http://api:5001`
- `http://weaviate:8080`
- `http://db:5432`

## 📝 참고 사항

- 원본 Dify 저장소: https://github.com/langgenius/dify
- 공식 문서: https://docs.dify.ai
- Docker Compose 버전: v2.15.1 이상 필요

## 🤝 기여

이 프로젝트는 커스텀 설정을 위한 것입니다.
원본 Dify 프로젝트에 기여하려면 공식 저장소를 참조하세요.

---

**Last Updated**: 2025-10-04
**Dify Version**: 1.9.1
