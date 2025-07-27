# Ktor + Compose Multiplatform Web 블로그 서비스 개발 지침 (단일 Git 레포 · **독립 2‑Project**) & 10‑Stage 커리큘럼

> **목표**
> 코틀린만으로 백엔드(Ktor)와 프론트엔드(Compose Multiplatform Web)를 구현하여, 가벼우면서 학습 효율이 높은 **블로그 플랫폼**을 완성한다.

---

## 1. 지침 프롬프트 (전반 지침)

### 언어 & 버전

* **Kotlin 1.9.22** (JVM 17, JS IR)

### 프로젝트 구조

하나의 **Git 레포** 안에 **서로 완전히 독립적인 Gradle 프로젝트** 두 개를 둡니다. 루트에는 README 정도만 두고, 빌드 관련 파일을 공유하지 않습니다.

```
blog-platform/                 ← Git 루트 (공통 README.md, .gitignore 등만)
├─ backend/                    ← Ktor + Exposed DAO (JVM)
│   ├─ gradlew / gradlew.bat   ← 각자 Wrapper
│   ├─ gradle/                 ← Wrapper JAR, 스크립트
│   ├─ settings.gradle.kts     ← backend 전용
│   ├─ build.gradle.kts        ← backend 전용
│   └─ src/main/kotlin …
└─ frontend/                   ← Compose Multiplatform Web (JS IR)
    ├─ gradlew / gradlew.bat
    ├─ gradle/
    ├─ settings.gradle.kts
    ├─ build.gradle.kts
    └─ src/jsMain/kotlin …
```

| 디렉터리       | 주요 기술스택                                                                                              | 빌드 스크립트                    | 기본 소스 디렉터리                               |
| ---------- | ---------------------------------------------------------------------------------------------------- | -------------------------- | ---------------------------------------- |
| `backend`  | Ktor 2.x + Netty, Exposed DAO (SQLite → PostgreSQL), Koin 3.x, kotlinx‑serialization, Kotlin Logging | **독립 Gradle KTS (JVM 17)** | `src/main/kotlin`, `src/test/kotlin`     |
| `frontend` | JetBrains Compose Multiplatform Web, Material 3 Theme, `compose-markdown`                            | **독립 Gradle KTS (JS IR)**  | `src/jsMain/kotlin`, `src/jsTest/kotlin` |

> **TIP** : 각 프로젝트는 자체 `gradlew`를 사용해 `./backend/gradlew build`, `./frontend/gradlew jsBrowserProductionWebpack`처럼 **완전히 분리**된 빌드·의존성 체계를 갖습니다. 루트에는 Gradle 관련 파일을 두지 않습니다.

### 공통 개발 지침

* **코드 품질** : ktlint + detekt, Conventional Commits
* **테스트** : Kotest (backend) / Compose UI tests (frontend)
* **CI** : GitHub Actions — 각 서브디렉터리별 **독립 Workflow**(또는 `paths` 필터)로 Gradle 캐싱·테스트·ktlint·Docker build 실행
* **배포** : 홈서버 Docker Compose (backend 8080, Nginx 80) + 포트포워딩
* **모니터링** : Micrometer + Prometheus, Grafana
* **라이브러리 크기 지침** : 개별 JAR < 5 MB, 프런트 번들 < 1.5 MB GZIP
* **보안** : 초기 공개 API → 추후 Admin JWT, HTTPS (Cloudflare Tunnel 권장)

---

## 2. 10‑Stage 커리큘럼

| Stage | 핵심 주제                     | (1) 이론 요점                                                                                   | (2) 실습 TODO                                                                |
| ----- | ------------------------- | ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **1** | 프로젝트 설계 & 스켈레톤            | - Git 루트 아래 `backend`, `frontend` 두 디렉터리 생성<br>- **각 디렉터리에서 `gradle init` → Kotlin DSL 선택** | • GitHub 첫 커밋<br>• backend & frontend **각자** `build.gradle.kts` 작성 후 빌드 통과 |
| **2** | DB 스키마 & Exposed DAO      | - Entity: `Post`, `User`, `Tag`, `PostTag`<br>- Flyway 마이그레이션                               | • Exposed DAO 구현<br>• testcontainers + SQLite integration test             |
| **3** | REST API (Posts)          | - Routing, Pagination, Validation<br>- Problem+JSON Error 모델                                | • `/posts` CRUD 구현<br>• Swagger (OpenAPI) 생성                               |
| **4** | 인증 & 권한 (옵션)              | - JWT 구조, Ktor Auth<br>- Bcrypt 패스워드                                                        | • `/auth/login`, `/auth/register` 구현<br>• Admin Role 미들웨어                  |
| **5** | 파일 업로드 & Static           | - Multipart Form‑Data, Content‑Type 검증<br>- Local File Storage → Nginx proxy                | • `/upload` 엔드포인트<br>• `/static/{fname}` 서빙                                |
| **6** | Compose Web 셋업            | - `compose.desktop.application` → Web target<br>- Routing(Navigation) & Material 3 Theme    | • Home 스크린, Post Card 컴포넌트 작성                                              |
| **7** | API 연동 & 상태관리             | - Ktor HttpClient MPP<br>- Coroutines + State Hoisting                                      | • `/posts` fetch → LazyColumn 렌더                                           |
| **8** | Markdown Editor & Preview | - `compose-markdown` 파서<br>- Split Pane (편집 vs 미리보기)                                        | • Admin 전용 글쓰기 UI<br>• Live Preview 구현                                     |
| **9** | CI/CD & Docker            | - GitHub Actions matrix: JVM + JS (서브디렉터리별 빌드)                                              |                                                                            |

* Docker Compose (backend + nginx + prometheus) | • `.github/workflows/backend.yml`, `.github/workflows/frontend.yml` 작성<br>• 서버에 `docker compose up -d` 배포 |
  \| **10** | 모니터링 & 확장 | - Micrometer Prometheus Registry<br>- Grafana dashboard import | • `/metrics` 노출<br>• Grafana alert rule 설정 |

---

## 3. 진척 관리 체크리스트

* [ ] Stage 1 완료 : backend · frontend 각자 빌드 성공
* [ ] Stage 2 완료 : DAO 테스트 통과
* [ ] Stage 3 완료 : `/posts` CRUD Swagger 확인
* [ ] Stage 4 완료 : JWT 발급 & 검증
* [ ] Stage 5 완료 : 이미지 업로드 동작
* [ ] Stage 6 완료 : Compose Web 빌드 & 브라우저 렌더
* [ ] Stage 7 완료 : Home → 상세 Post 네비게이션
* [ ] Stage 8 완료 : 실시간 Markdown 프리뷰
* [ ] Stage 9 완료 : CI 파이프라인 pass, Docker deploy
* [ ] Stage 10 완료 : Grafana 대시보드 실시간 데이터 표시

---

## 4. 참고 링크

* **JetBrains Compose Multiplatform Docs** : [https://compose.multiplatform.io/](https://compose.multiplatform.io/)
* **Ktor Docs** : [https://ktor.io/](https://ktor.io/)
* **Exposed DAO Guide** : [https://github.com/JetBrains/Exposed](https://github.com/JetBrains/Exposed)
* **Compose Markdown** : [https://github.com/arkivanov/compose-markdown](https://github.com/arkivanov/compose-markdown)
* **Micrometer Prometheus** : [https://micrometer.io/docs/registry/prometheus](https://micrometer.io/docs/registry/prometheus)

---

### 🔄 다음 단계

1. **Stage 1** : `backend`, `frontend` 디렉터리에서 **각자** Gradle 스켈레톤을 만들고 첫 빌드를 통과하세요.
2. 강의 · 실습 중 의문이 생기면 각 Stage 끝에 질문을 남겨 주세요.
3. 완료 체크리스트를 갱신하며 진행 상황을 추적하세요.

즐거운 코틀린 풀스택 여정이 되길 바랍니다! 🚀
