# QuerySentinel

데이터베이스 성능 회귀를 자동으로 검증하는 플랫폼을 목표로 하는 프로젝트입니다.
.
## Tech Stack

- Backend: Spring Boot 4.0.9, Java 21, Gradle
- Frontend: React 19, Vite
- Database: MySQL 8.4
- ORM: Spring Data JPA / Hibernate
- Container: Docker Compose

## Project Structure

```text
QuerySentinel/
├── backend/                 # Spring Boot
├── frontend/                # React + Vite
├── docker/
│   └── docker-compose.yml   # MySQL 8.4
├── .env.example             # 환경변수 예시
└── .gitignore
```

## 1. Prerequisites

- Java 21
- IntelliJ IDEA
- Node.js / npm
- Docker Desktop
- Git

## 2. Clone

```bash
git clone https://github.com/zuzusnasna/QuerySentinel.git
cd QuerySentinel
```

## 3. 환경변수 설정

실제 비밀번호가 포함된 `.env`는 Git에 올리지 않습니다.

프로젝트 루트에서 `.env.example`을 복사하여 `.env`를 생성합니다.

### Windows PowerShell

```powershell
Copy-Item .env.example .env
```

`.env`를 열고 본인이 사용할 비밀번호로 변경합니다.

```dotenv
MYSQL_ROOT_PASSWORD=사용할_ROOT_비밀번호
MYSQL_PASSWORD=사용할_APP_비밀번호
```

`MYSQL_PASSWORD`는 Spring Boot가 사용하는 `qs_app` 계정의 비밀번호입니다.

> 처음 Docker MySQL을 생성할 때 `.env`의 값을 기준으로 `qs_app` 계정이 생성됩니다.
> 이미 MySQL 볼륨이 생성된 상태에서 `.env`의 비밀번호만 변경하면 기존 DB 계정 비밀번호는 자동으로 변경되지 않습니다.

## 4. MySQL 실행

프로젝트 루트에서 실행합니다.

```powershell
docker compose -f docker/docker-compose.yml up -d
```

상태 확인:

```powershell
docker compose -f docker/docker-compose.yml ps
```

정상적으로 실행되면 다음과 같이 사용합니다.

| 항목 | 값 |
|---|---|
| MySQL Host | `localhost` |
| MySQL Port | `3308` |
| Database | `querysentinel` |
| User | `qs_app` |
| Password | `.env`의 `MYSQL_PASSWORD` |
| Root Password | `.env`의 `MYSQL_ROOT_PASSWORD` |

MySQL 컨테이너 내부 포트는 `3306`이고, 로컬 PC에서는 `3308`로 접근합니다.

## 5. Backend 실행

Backend는 다음 설정을 사용합니다.

```properties
spring.datasource.url=jdbc:mysql://localhost:3308/querysentinel
spring.datasource.username=qs_app
spring.datasource.password=${MYSQL_PASSWORD}
server.port=8081
```

즉, Backend는 다음 주소에서 실행됩니다.

```text
http://localhost:8081
```

### IntelliJ에서 실행하는 경우

`backend/src/main/java/.../BackendApplication.java`를 열고 IntelliJ의 초록색 실행 버튼으로 실행합니다.

Run Configuration의 **Environment variables**에 다음을 추가합니다.

```text
MYSQL_PASSWORD=본인의_.env와_동일한_비밀번호
```

비밀번호는 GitHub에 커밋하지 않습니다.

> IntelliJ의 초록색 실행 버튼은 프로젝트 루트의 `.env` 파일을 자동으로 읽지 않습니다. 따라서 현재 프로젝트에서는 IntelliJ Run Configuration에 `MYSQL_PASSWORD`를 환경변수로 등록해야 합니다.

### Gradle로 실행하는 경우

```powershell
cd backend
.\gradlew.bat bootRun
```

Gradle 실행 시에도 `MYSQL_PASSWORD` 환경변수가 필요합니다.

## 6. Frontend 실행

```powershell
cd frontend
npm install
npm run dev
```

Vite 기본 개발 서버 주소:

```text
http://localhost:5173
```

## 7. 전체 실행 순서

처음 프로젝트를 받은 팀원은 다음 순서로 실행합니다.

```text
1. Git clone
      ↓
2. .env.example → .env 복사
      ↓
3. .env의 MYSQL_ROOT_PASSWORD / MYSQL_PASSWORD 설정
      ↓
4. Docker Desktop 실행
      ↓
5. MySQL 컨테이너 실행
      ↓
6. IntelliJ에서 MYSQL_PASSWORD 환경변수 등록
      ↓
7. BackendApplication 실행
      ↓
8. frontend에서 npm install
      ↓
9. npm run dev
```

## 8. 포트 정리

| 서비스 | 로컬 포트 |
|---|---:|
| Frontend (Vite) | `5173` |
| Backend (Spring Boot) | `8081` |
| MySQL | `3308` |

다른 프로그램이 위 포트를 사용하고 있다면 충돌이 발생할 수 있습니다.

MySQL 포트를 변경할 경우 `docker/docker-compose.yml`의 호스트 포트와 `backend/src/main/resources/application.properties`의 JDBC URL 포트를 **같이 변경해야 합니다.**

예:

```yaml
ports:
  - "3309:3306"
```

```properties
spring.datasource.url=jdbc:mysql://localhost:3309/querysentinel?serverTimezone=Asia/Seoul&characterEncoding=UTF-8
```

## 9. MySQL 접속 확인

컨테이너 내부에서 `qs_app` 계정으로 접속할 수 있습니다.

```powershell
docker exec -it querysentinel-mysql mysql -uqs_app -p querysentinel
```

비밀번호는 `.env`의 `MYSQL_PASSWORD`와 동일해야 합니다.

## 10. 중지

```powershell
docker compose -f docker/docker-compose.yml down
```

> `down -v`는 MySQL 데이터 볼륨까지 삭제할 수 있으므로 개발 데이터가 필요한 경우 사용하지 않습니다.
