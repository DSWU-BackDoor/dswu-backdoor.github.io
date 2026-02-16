
# 🚪 BackDoor 홈페이지 운영 매뉴얼

### 1️⃣ 시스템 구조

| 영역       | 설명                             |
| -------- | ------------------------------ |
| Backend  | Spring Boot (Fly.io 배포)        |
| Frontend | 정적 HTML / JS (GitHub Pages 배포) |
| CI/CD    | GitHub Actions 자동 배포           |
| 인증       | JWT + BCrypt (환경변수 기반)         |

### 2️⃣ 레포지토리 구조

```
backend/              # API 서버 (Spring Boot)
frontend/             # 정적 페이지 파일
scripts/              # 초기 세팅용 (운영 중 사용 안 함)
.github/workflows/    # GitHub Actions 배포 설정
```

### 3️⃣ 수정 및 배포 방법

**① 코드 수정**

필요한 파일 수정 후
```bash
git add .
git commit -m "수정 내용 설명"
git push
```

**② 배포 확인**

GitHub → **Actions** 탭 확인
* 프론트 배포: `pages-frontend.yml`
* 백엔드 배포: `fly-backend.yml`

모두 ✅ 성공이어야 실제 서비스에 반영됨.

### 4️⃣ 관리자 비밀번호 변경 방법

**① BCrypt 해시 생성**

```java
new org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder()
    .encode("새 비밀번호");
```

출력된 해시값 복사.

**② Fly 환경변수 교체**

```bash
fly secrets set ADMIN_PASSWORD_HASH='생성된_해시값' --app <FLY_APP>
```

필요 시:

```bash
fly secrets set JWT_SECRET='기존값' --app <FLY_APP>
```

→ 설정 즉시 자동 재배포됨.

**③ 로그인 테스트**

```bash
curl -X POST https://<FLY_APP>.fly.dev/api/admin/auth/login \
  -H "Content-Type: application/json" \
  -d '{"password":"새 비밀번호"}'
```

* `200 + accessToken` → 성공
* `401` → 해시 불일치

### ⚠️ 운영 시 주의사항

* GitHub Actions 실패 시 배포 반영 안 됨
* Secrets 값은 Git에 커밋하지 말 것
* Fly 로그 확인: `fly logs --app <FLY_APP>`
  
### 📌 서비스 주소
* 홈페이지: `https://dswu-backdoor.github.io/`
* 백엔드 Health Check: `https://<FLY_APP>.fly.dev/api/health`
