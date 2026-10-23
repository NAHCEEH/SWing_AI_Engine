# SWing AI Engine — Backend

FastAPI와 PostgreSQL 기반의 팀 매칭 백엔드입니다. 학생·교수 인증, 수업 관리, 수업별 설문, 팀 편성을 위한 API를 제공하며 수강 정보와 매칭 결과를 데이터베이스에 저장합니다.

## 주요 기능

- **인증**: 회원가입, 로그인, JWT 기반 사용자 인증
- **수업 관리**: 교수의 수업 관리, 참여 코드를 통한 학생 등록, 수강생 명단 조회
- **설문 관리**: 수업별 역할, 스킬, 작업 성향, 가용 시간 제출 및 조회
- **팀 매칭**: 가중치와 제약 조건에 따른 팀 편성, 편성 전 조건 점검
- **결과 저장**: 매칭 실행 내역, 팀 구성, 팀원 배정 정보 저장
- **프리셋**: 기본 매칭 설정 조회

## 기술 스택

| 기술 | 용도 |
| --- | --- |
| Python / FastAPI | HTTP API 및 요청 유효성 검사 |
| Pydantic | 요청·응답 모델 |
| SQLAlchemy 2.0 | 데이터베이스 ORM 및 세션 관리 |
| PostgreSQL / psycopg | 데이터 저장 및 연결 |
| python-jose / passlib(bcrypt) | JWT 인증 및 비밀번호 해싱 |
| pytest | 단위 테스트 |

매칭 엔진은 Python으로 구현되어 있으며 데이터베이스 없이 단독으로 테스트할 수 있습니다.

## 폴더 구조

```text
backend/
├── app/
│   ├── config.py             # 환경변수 설정
│   ├── database.py           # DB 연결, 세션, get_db 의존성
│   ├── models.py             # ORM 모델
│   ├── security.py           # 비밀번호 해싱 및 JWT 인증
│   ├── schemas.py            # 요청·응답 모델
│   ├── matching.py           # 기존 그리디 매칭 엔진
│   ├── main.py               # 앱 진입점, CORS, 라우터 등록
│   ├── engine/               # TeamFit 매칭 엔진
│   └── routers/
│       ├── auth.py           # 인증 API
│       ├── classes.py        # 수업 및 수강생 API
│       ├── surveys.py        # 설문 API
│       ├── matching.py       # 기존 매칭 및 프리셋 API
│       └── matching_v2.py    # 사전 점검 및 TeamFit 매칭 API
├── db/
│   ├── schema.sql            # PostgreSQL 스키마 및 기본 프리셋
│   └── schema.mysql.sql      # MySQL 스키마 참고 파일
├── scripts/
│   └── engine_demo.py        # 합성 데이터 기반 엔진 시연
├── tests/
│   ├── test_matching.py
│   ├── test_match_guards.py
│   ├── test_engine_v2.py
│   └── test_engine_port.py
├── requirements.txt
├── pytest.ini
├── .env.example
├── TESTING.md
└── README.md
```

아래 실행 안내는 PostgreSQL을 기준으로 합니다.

## 설치 및 실행

Python과 PostgreSQL이 설치되어 있고 PostgreSQL 서버가 실행 중이어야 합니다. 모든 명령은 저장소의 `backend/` 폴더에서 실행합니다.

### 1. 가상환경 및 패키지 설치

**Windows / PowerShell**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
Copy-Item .env.example .env
```

**macOS / Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cp .env.example .env
```

### 2. 데이터베이스 준비

다음은 `.env.example`의 접속 정보에 맞춘 로컬 개발용 예시입니다. `postgres` 관리자 계정으로 접속할 수 있어야 하며, `teammatch` 사용자와 데이터베이스가 이미 있다면 생성 명령은 생략합니다.

```bash
psql -U postgres -c "CREATE USER teammatch WITH PASSWORD 'teammatch';"
psql -U postgres -c "CREATE DATABASE teammatch OWNER teammatch;"
psql -h localhost -U teammatch -d teammatch -f db/schema.sql
```

기존 데이터베이스를 사용한다면 해당 접속 정보를 `.env`의 `DATABASE_URL`에 입력합니다.

### 3. 환경변수 설정

복사한 `.env` 파일을 실행 환경에 맞게 수정합니다.

```dotenv
DATABASE_URL=postgresql+psycopg://teammatch:teammatch@localhost:5432/teammatch
JWT_SECRET=change-me-in-production
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=1440
CORS_ORIGINS=["http://localhost:5500","http://127.0.0.1:5500"]
```

| 변수 | 설명 |
| --- | --- |
| `DATABASE_URL` | PostgreSQL 사용자, 비밀번호, 호스트, 포트, DB 이름 |
| `JWT_SECRET` | JWT 서명 키. 운영 환경에서는 별도의 비밀키로 변경 |
| `JWT_ALGORITHM` | JWT 서명 알고리즘 |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | 액세스 토큰 유효 시간(분) |
| `CORS_ORIGINS` | API 요청을 허용할 프론트엔드 주소 목록 |

### 4. 서버 실행

가상환경이 활성화된 상태에서 실행합니다.

```bash
python -m uvicorn app.main:app --reload
```

- 서버 주소: [http://127.0.0.1:8000](http://127.0.0.1:8000)
- 대화형 API 문서: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

요청 본문과 응답 형식은 대화형 API 문서에서 확인할 수 있습니다.

## 주요 API

아래 경로의 `{class_id}`는 수업 ID입니다.

| 기능 | 메서드 | 경로 |
| --- | --- | --- |
| 회원가입 | `POST` | `/api/auth/signup` |
| 로그인 | `POST` | `/api/auth/login` |
| 내 정보 조회 | `GET` | `/api/auth/me` |
| 수업 생성 | `POST` | `/api/classes` |
| 수업 목록 조회 | `GET` | `/api/classes` |
| 참여 코드로 수업 등록 | `POST` | `/api/classes/join` |
| 수강생 및 설문 제출 상태 조회 | `GET` | `/api/classes/{class_id}/roster` |
| 설문 제출 | `PUT` | `/api/classes/{class_id}/survey` |
| 설문 조회 | `GET` | `/api/classes/{class_id}/survey` |
| 기본 프리셋 조회 | `GET` | `/api/presets` |
| 기존 엔진으로 팀 편성 | `POST` | `/api/classes/{class_id}/match` |
| TeamFit 조건 사전 점검 | `POST` | `/api/classes/{class_id}/precheck` |
| TeamFit 엔진으로 팀 편성 | `POST` | `/api/classes/{class_id}/match-v2` |

## 데이터 모델

- **사용자**: `users`의 `role`로 학생과 교수를 구분합니다. 역할별 전용 컬럼은 `NULL`을 허용합니다.
- **수업별 설문**: 수강 내역을 기준으로 `survey_responses`, `survey_skills`, `survey_availability`에 저장합니다. 같은 학생도 수업마다 다른 설문을 제출할 수 있습니다.
- **가용 시간**: 선택한 시간 슬롯마다 한 행을 저장합니다. `day`는 월요일 `0`부터 일요일 `6`까지이며, `block`은 프론트엔드 시간 블록 인덱스와 대응합니다.
- **기본 프리셋**: `owner_id IS NULL`, `is_builtin=true`인 공유 설정을 `db/schema.sql`의 시드 데이터로 제공합니다.
- **매칭 결과**: 실행 내역은 `match_runs`, 팀 정보는 `teams`, 팀원 배정은 `team_members`에 저장합니다.

## 매칭 엔진

### 기존 엔진 — `app/matching.py`

`POST /api/classes/{class_id}/match`에서 사용하는 그리디 방식의 엔진입니다.

```python
form_teams(candidates, team_size, weights, hard)
```

1. 학생 수와 팀 크기로 팀 수를 계산합니다: `ceil(n / team_size)`.
2. 희귀한 역할을 먼저 배정한 뒤, 남은 학생을 팀 점수가 가장 크게 개선되는 팀에 배정합니다.
3. 설정된 가중치에 따라 팀별 조화도를 0~100점으로 계산합니다.

| 평가 축 | 내용 |
| --- | --- |
| `major` | 전공 다양성. `same_major` 조건 사용 시 전공 동일성 |
| `role` | 서로 다른 역할의 충족도 |
| `time` | 가용 시간의 겹침 |
| `trait` | 작업 성향의 유사도 |

### TeamFit 엔진 — `app/engine/`

`precheck`와 `match-v2` API에서 사용하는 매칭 엔진입니다. 엔진 내부 구현은 v1.1이며 API 경로는 `match-v2`를 사용합니다.

- **사전 점검**: 조건 충족 가능 여부와 항목별 점검 결과를 반환합니다. 충족할 수 없는 조건은 매칭 API에서 HTTP `422` 응답과 완화 안내를 제공합니다.
- **태그 해석**: `개발`, `4학년` 등 역할·학년 태그를 인식합니다. 지원하지 않는 태그는 점검 실패 항목으로 알립니다.
- **팀 크기 배분**: 기준 팀 수는 `floor(n / team_size)`이며 남는 학생을 앞 팀부터 추가 배정합니다. 예를 들어 46명을 기준 크기 4명으로 편성하면 5명 팀 2개와 4명 팀 9개가 됩니다.
- **최적화**: 담금질 탐색을 사용하며 반복 횟수는 학생 수 × 400회, 전체 계산 시간 상한은 20초입니다. `MatchV2Request.starts`로 탐색 시작 수를 1~5 범위에서 지정할 수 있으며 기본값은 3입니다.
- **결과 설명**: 목적함수 `J`, 평균·최저 조화도, 하드 조건 위반 수, 교환 횟수, 팀별 결과, 조화도 추이, 주관식 해석을 제공합니다.
- **평가 기능**: `scoring.py`는 채점, `explain.py`는 팀 설명과 `evaluate`를 담당합니다.

엔진 구성과 계산식은 [엔진 README](app/engine/README.md)를 참고하세요. 두 엔진 모두 휴리스틱 방식이며 최적해를 보장하지 않습니다. TeamFit의 세부 제약 조건을 모두 영구 저장하려면 기존 불리언 플래그 외에 JSON 등 저장 구조 확장이 필요합니다.

## 테스트

`backend/`에서 실행합니다. 엔진 단위 테스트에는 실행 중인 데이터베이스와 `.env`가 필요하지 않습니다.

```bash
python -m pytest -q
```

`pytest.ini`에 import 경로와 테스트 폴더가 지정되어 있어 `pytest -q`로도 실행할 수 있습니다.

| 테스트 파일 | 문서 기준 개수 | 검증 내용 |
| --- | ---: | --- |
| `tests/test_matching.py` | 5 | 기존 그리디 엔진, 팀 배정 및 점수 |
| `tests/test_match_guards.py` | 3 | 매칭 라우터의 중복 배정 방어 |
| `tests/test_engine_v2.py` | 9 | 점수 범위, 패널티, 전원 배정, 시드 재현성, 사전 점검, 주관식 해석 |
| `tests/test_engine_port.py` | 17 | TeamFit v1.1의 팀 수 규칙, 태그, 역할 배정, 채점, 제약 조건, 완화안, 평가 |
| `tests/test_llm_rules.py` | 16 | 교수 메모 LLM 해석기: 응답 검증, 지원 안 되는 요구 표시, 실패 시 규칙 기반 복귀, 캐시, 요청 형식 (실제 LLM 호출 없음) |
| **합계** | **50** | |

설치 및 API 흐름별 검증 방법은 [TESTING.md](TESTING.md)를 참고하세요. 위 테스트 목록은 인증·권한·수업 흐름 전체에 대한 API 통합 테스트를 포함하지 않습니다.

## 엔진 시연

데이터베이스 없이 합성 수강생 데이터로 사전 점검, 팀 편성, 조건 검산, 팀별 설명과 재평가를 확인할 수 있습니다.

```bash
python scripts/engine_demo.py
python scripts/engine_demo.py --n 90
python scripts/engine_demo.py --no-note
```

기본 시연은 수강생 45명과 교수 메모를 사용합니다. `--n`은 수강생 수를 지정하고 `--no-note`는 교수 메모 없이 실행합니다.
