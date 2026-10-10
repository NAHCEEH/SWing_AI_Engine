From e4054b22628a236ef83413a49758f32813d0bcb4 Mon Sep 17 00:00:00 2001
From: =?UTF-8?q?=EC=A0=95=ED=9D=AC=EC=B0=AC?= <dc07080@hanyang.ac.kr>
Date: Sat, 10 Oct 2026 13:19:24 +0900
Subject: [PATCH 1/2] =?UTF-8?q?=ED=85=8C=EC=8A=A4=ED=8A=B8=20=EB=AC=B8?=
 =?UTF-8?q?=EC=84=9C=20=EA=B0=B1=EC=8B=A0:=20AI=20=EC=97=94=EC=A7=84=20v1.?=
 =?UTF-8?q?1=20=EB=B0=98=EC=98=81,=20pytest=20=EC=8B=A4=ED=96=89=20?=
 =?UTF-8?q?=EB=B0=A9=EB=B2=95=20=EC=88=98=EC=A0=95?=
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit

- TESTING.md: 6장을 AI 엔진 v1.1 기준으로 갱신 (태그·팀 수 규칙, 속도),
  6-3 테스트 실행을 `python -m pytest -q`로 고치고 파일별 34개 표 추가,
  6-4 엔진 시연(scripts/engine_demo.py) 추가, macOS/Linux 설치 명령 추가,
  부록 B의 '테스트 미검증'·'.gitignore 없음' 항목 갱신
- README.md: 테스트 파일 설명과 실행 방법, v2 엔진 절에 v1.1 교체 안내 추가
- pytest.ini 추가: `pytest`만 실행해도 `from app ...` import가 되도록 (전에는 4개 파일 수집 실패)
- .env.example 추가: 문서가 가리키던 파일이 없어서 config.py 기본값으로 생성

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01M7wfP91feLoQnFqvNdTruV
---
 backend/.env.example |  7 ++++++
 backend/README.md    | 22 ++++++++++++++---
 backend/TESTING.md   | 58 +++++++++++++++++++++++++++++++++++++++-----
 backend/pytest.ini   |  4 +++
 4 files changed, 82 insertions(+), 9 deletions(-)
 create mode 100644 backend/.env.example
 create mode 100644 backend/pytest.ini

diff --git a/backend/.env.example b/backend/.env.example
new file mode 100644
index 0000000..abcfa77
--- /dev/null
+++ b/backend/.env.example
@@ -0,0 +1,7 @@
+# 복사해서 .env로 만든 뒤 값을 채웁니다: cp .env.example .env
+# 값이 없으면 app/config.py의 기본값을 씁니다.
+DATABASE_URL=postgresql+psycopg://teammatch:teammatch@localhost:5432/teammatch
+JWT_SECRET=change-me-in-production
+JWT_ALGORITHM=HS256
+ACCESS_TOKEN_EXPIRE_MINUTES=1440
+CORS_ORIGINS=["http://localhost:5500","http://127.0.0.1:5500"]
diff --git a/backend/README.md b/backend/README.md
index 5b92a79..f13e063 100644
--- a/backend/README.md
+++ b/backend/README.md
@@ -28,7 +28,7 @@ psql -U postgres -d teammatch -f db/schema.sql
 copy .env.example .env   # 복사 후 값 수정
 
 uvicorn app.main:app --reload
-(위 코드 블록의 명령어 주석은 원문 코드를 번역한 것입니다.)   대화형 API 문서: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)   엔진 테스트 실행: pytest (backend/ 폴더에서 실행).   프론트엔드 연동 매핑 (How it maps to the frontend)프론트엔드 (index.html)백엔드authRole 학생/교수, 로그인/회원가입   POST /api/auth/signup, POST /api/auth/login, GET /api/auth/me   state.prof.classes[], 참여 코드 c.join   POST /api/classes, GET /api/classes, POST /api/classes/join   ROSTER (학생 명단) + 제출 상태   GET /api/classes/{id}/roster   state.user 역할 / 스킬 / 성향 / 일정   PUT/GET /api/classes/{id}/survey   setup.weights + hard + 프리셋   POST /api/classes/{id}/match, GET /api/presets   "AI 팀매칭" 실행 + state.prof.result   매칭 결과는 match_runs + teams + team_members에 저장됨   데이터 모델 참고 사항하나의 users 테이블을 사용하며 role(역할)로 구분합니다. 학생 전용 및 교수 전용 컬럼은 Null 값을 허용(nullable)합니다.   설문 데이터(Survey signals)는 수강 내역(enrollment)을 기준으로 survey_responses (+ survey_skills, survey_availability)에 저장되므로, 학생은 수업마다 다르게 답변할 수 있습니다.   가용 시간(Availability)은 프론트엔드의 "day-block" 그리드를 선택된 슬롯당 한 줄씩 데이터로 저장합니다. day 0=월 .. 6=일이며, block은 프론트엔드의 BLOCKS[] 인덱스와 일치합니다.   owner_id IS NULL (is_builtin=true)인 프리셋(Presets)은 schema.sql 하단에 시드 데이터로 제공되는 공유 기본값(프론트엔드의 ps0..ps4)입니다.   매칭 엔진 (app/matching.py)   form_teams(candidates, team_size, weights, hard) 함수는 그리디(greedy) 밸런싱 휴리스틱(탐색 기법)을 사용합니다:   팀 수 = ceil(n / team_size)로 계산합니다.   가장 희귀한 역할을 먼저 각 팀에 하나씩 배정한 후(seed), 남은 학생들은 추가 시 팀을 가장 크게 개선할 수 있는 곳에 그리디 방식으로 배정합니다(가능한 한 하드 제약 조건을 준수함).   각 팀은 교수가 설정한 1~5점 가중치를 바탕으로 4가지 축을 혼합하여 0~100점의 조화도 점수(fit score)를 받습니다:   major — 전공 다양성 (또는 same_major 하드 제약 조건이 켜져 있을 경우 동일성)   role — 서로 다른 역할의 충족도   time — 가용한 일정 슬롯의 겹침 정도   trait — 팀원 간 작업 스타일(성향)의 유사도   이것은 정확한 최적화 알고리즘이 아니라 휴리스틱 탐색으로, 설명 가능하고 좋은 팀을 빠르게 구성해 줍니다. 만약 나중에 수학적으로 완벽한 최적의 배정이 필요하다면, API나 DB 계층을 건드릴 필요 없이 이 모듈 하나만 ILP(정수계획법)/제약 조건 솔버(solver)로 교체하면 됩니다. 
+(위 코드 블록의 명령어 주석은 원문 코드를 번역한 것입니다.)   대화형 API 문서: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)   테스트 실행: backend/ 폴더에서 python -m pytest -q (34개, 자세한 방법은 TESTING.md).   프론트엔드 연동 매핑 (How it maps to the frontend)프론트엔드 (index.html)백엔드authRole 학생/교수, 로그인/회원가입   POST /api/auth/signup, POST /api/auth/login, GET /api/auth/me   state.prof.classes[], 참여 코드 c.join   POST /api/classes, GET /api/classes, POST /api/classes/join   ROSTER (학생 명단) + 제출 상태   GET /api/classes/{id}/roster   state.user 역할 / 스킬 / 성향 / 일정   PUT/GET /api/classes/{id}/survey   setup.weights + hard + 프리셋   POST /api/classes/{id}/match, GET /api/presets   "AI 팀매칭" 실행 + state.prof.result   매칭 결과는 match_runs + teams + team_members에 저장됨   데이터 모델 참고 사항하나의 users 테이블을 사용하며 role(역할)로 구분합니다. 학생 전용 및 교수 전용 컬럼은 Null 값을 허용(nullable)합니다.   설문 데이터(Survey signals)는 수강 내역(enrollment)을 기준으로 survey_responses (+ survey_skills, survey_availability)에 저장되므로, 학생은 수업마다 다르게 답변할 수 있습니다.   가용 시간(Availability)은 프론트엔드의 "day-block" 그리드를 선택된 슬롯당 한 줄씩 데이터로 저장합니다. day 0=월 .. 6=일이며, block은 프론트엔드의 BLOCKS[] 인덱스와 일치합니다.   owner_id IS NULL (is_builtin=true)인 프리셋(Presets)은 schema.sql 하단에 시드 데이터로 제공되는 공유 기본값(프론트엔드의 ps0..ps4)입니다.   매칭 엔진 (app/matching.py)   form_teams(candidates, team_size, weights, hard) 함수는 그리디(greedy) 밸런싱 휴리스틱(탐색 기법)을 사용합니다:   팀 수 = ceil(n / team_size)로 계산합니다.   가장 희귀한 역할을 먼저 각 팀에 하나씩 배정한 후(seed), 남은 학생들은 추가 시 팀을 가장 크게 개선할 수 있는 곳에 그리디 방식으로 배정합니다(가능한 한 하드 제약 조건을 준수함).   각 팀은 교수가 설정한 1~5점 가중치를 바탕으로 4가지 축을 혼합하여 0~100점의 조화도 점수(fit score)를 받습니다:   major — 전공 다양성 (또는 same_major 하드 제약 조건이 켜져 있을 경우 동일성)   role — 서로 다른 역할의 충족도   time — 가용한 일정 슬롯의 겹침 정도   trait — 팀원 간 작업 스타일(성향)의 유사도   이것은 정확한 최적화 알고리즘이 아니라 휴리스틱 탐색으로, 설명 가능하고 좋은 팀을 빠르게 구성해 줍니다. 만약 나중에 수학적으로 완벽한 최적의 배정이 필요하다면, API나 DB 계층을 건드릴 필요 없이 이 모듈 하나만 ILP(정수계획법)/제약 조건 솔버(solver)로 교체하면 됩니다. 
 
 
 ---
@@ -105,6 +105,12 @@ uvicorn app.main:app --reload
 #### `tests/test_matching.py` — 매칭 엔진 단위 테스트
 - DB 없이 순수하게 매칭 엔진만 검증합니다: 팀 수·배정 정확성, 점수 범위(0~100), `same_major`/역할 가중치 동작, 빈 후보 처리 등.
 
+#### `tests/test_engine_v2.py`, `tests/test_engine_port.py`, `tests/test_match_guards.py`
+- `test_engine_v2.py`: AI 엔진이 라우터와 맞춰야 하는 계약(점수 범위, J 패널티, 전원 배정, 시드 재현, 사전 점검, 주관식 해석).
+- `test_engine_port.py`: AI 엔진 v1.1 동작(팀 수 규칙, `개발`·`N학년` 태그, 역할 배정, 하드 조건 준수, 완화안, evaluate).
+- `test_match_guards.py`: 매칭 라우터의 중복 배정 방어.
+- 실행: `backend/`에서 `python -m pytest -q` → 34개 통과. 파일별 설명은 `TESTING.md` 6-3.
+
 #### `requirements.txt` / `.env.example`
 - `requirements.txt`: 필요한 파이썬 패키지 목록(FastAPI, SQLAlchemy, psycopg, JWT/bcrypt 등).
 - `.env.example`: 환경변수 예시 파일. 복사해서 `.env`로 만든 뒤 값을 채웁니다.
@@ -118,6 +124,12 @@ uvicorn app.main:app --reload
 위에 더 정교한 최적화 엔진을 `app/engine/` 패키지로 추가했습니다. 기존 엔진은
 그대로 두고, v2는 별도 엔드포인트로 제공합니다.
 
+> **2026-10-10 업데이트**: `app/engine` 내부를 AI 엔진 v1.1로 교체했습니다.
+> 엔드포인트·요청/응답 형식·라우터가 부르는 함수 이름은 그대로입니다.
+> 최신 구성과 공식은 [app/engine/README.md](app/engine/README.md)에 있고,
+> 아래 설명과 달라진 점은 `scoring.py`(빠른 채점기)·`explain.py`(팀 설명·evaluate) 추가,
+> `개발`·`N학년` 태그 인식, 팀 수 규칙(남는 학생은 큰 팀으로), 담금질 반복(학생 수 × 400, 전체 20초 상한)입니다.
+
 ### 구성 (app/engine/)
 
 | 파일 | 역할 |
@@ -170,13 +182,17 @@ uvicorn app.main:app --reload
 ### 테스트
 
 `tests/test_engine_v2.py` — 소프트 점수 범위, J 패널티, 전원 배정, 시드 결정성,
-사전 점검 불가 판정, 주관식 해석을 DB 없이 검증합니다. `pytest`로 실행.
+사전 점검 불가 판정, 주관식 해석을 DB 없이 검증합니다.
+`tests/test_engine_port.py` — AI 엔진 v1.1 동작을 검증합니다 (17개).
+`backend/`에서 `python -m pytest -q`로 실행하면 전체 34개가 통과합니다.
+합성 데이터로 엔진만 돌려 보려면 `python scripts/engine_demo.py`.
 
 ### 참고 / 한계
 
 - v2 결과도 기존 `match_runs`/`teams`/`team_members` 테이블에 저장합니다.
   단, 하드 제약은 현재 테이블의 불리언 플래그로만 매핑되므로, 상한값·지정 쌍 등
   **세부 조건을 그대로 저장**하려면 전용 컬럼(JSON 등) 추가가 필요합니다.
-- SA 반복 횟수(기본 9,000)와 best-of-N(기본 3)은 `MatchV2Request`에서 조정 가능.
+- best-of-N(기본 3)은 `MatchV2Request.starts`(1~5)로 조정합니다. SA 반복은 v1.1부터
+  학생 수 × 400회로 자동 결정되고, 전체 계산은 20초를 넘지 않습니다.
 - 이 엔진은 휴리스틱이라 최적해를 보장하지 않습니다. 정확해가 필요하면
   `optimizer.py` 를 ILP/제약 솔버로 교체할 수 있습니다.
diff --git a/backend/TESTING.md b/backend/TESTING.md
index 2c225ac..9c14d56 100644
--- a/backend/TESTING.md
+++ b/backend/TESTING.md
@@ -13,6 +13,8 @@
 
 uvicorn을 바로 실행하면 패키지가 없어서 실패합니다. 먼저 설치하세요.
 
+Windows (PowerShell):
+
 ```powershell
 cd <프로젝트 경로>\backend
 python -m venv .venv
@@ -21,6 +23,18 @@ pip install -r requirements.txt
 copy .env.example .env      # 그리고 DATABASE_URL 등 값 확인/수정
 ```
 
+macOS / Linux (터미널):
+
+```bash
+cd <프로젝트 경로>/backend
+python3 -m venv .venv
+source .venv/bin/activate
+pip install -r requirements.txt
+cp .env.example .env        # 그리고 DATABASE_URL 등 값 확인/수정
+```
+
+> 엔진 단위 테스트(6-3)와 시연(6-4)만 돌릴 때는 DB와 `.env`가 필요 없습니다.
+
 `.env`의 `DATABASE_URL`이 실제 PostgreSQL 접속 정보와 맞는지 꼭 확인하세요.
 (기본값: `postgresql+psycopg://teammatch:teammatch@localhost:5432/teammatch`)
 
@@ -157,7 +171,10 @@ uvicorn app.main:app --reload
 
 ---
 
-## 6. v2 엔진 테스트 (TeamFit) — 팀원 메모에 없던 신규
+## 6. AI 엔진 테스트 (TeamFit) — 2026-10-10 v1.1로 갱신
+
+`app/engine` 내부가 AI 엔진 v1.1로 바뀌었습니다. 엔드포인트와 요청·응답 형식은
+그대로이고, 엔진 구성은 [app/engine/README.md](app/engine/README.md)에 있습니다.
 
 ### 6-1. 실행 전 점검 — `POST /api/classes/{class_id}/precheck`
 ```json
@@ -169,17 +186,44 @@ uvicorn app.main:app --reload
 }
 ```
 - 조건 충족 가능 여부(feasible)와 항목별 점검 결과를 돌려줍니다.
+- v1.1부터 태그 `개발`(프론트엔드·백엔드·데이터 희망자)과 `N학년`(예: `4학년`)을
+  인식합니다. 어휘에 없는 태그는 조용히 무시하지 않고 실패 항목으로 알려 줍니다.
+- 팀 수 = ⌊학생 수 ÷ team_size⌋, 남는 학생은 앞 팀부터 1명씩 더합니다
+  (46명·4명 → `5명×2 + 4명×9`).
 
 ### 6-2. v2 매칭 실행 — `POST /api/classes/{class_id}/match-v2`
 - 같은 body 사용. 응답에 목적함수 J, 평균/최저 조화도, 하드위반 수,
   교환 횟수, 팀별 결과(최저 팀 표시), 조화도 추이, 주관식 해석이 담깁니다.
 - 점검이 불가능(feasible=false)하면 422로 막고 완화 안내를 줍니다.
+- 45명 기준 약 1초 안에 끝납니다 (이전 v2 엔진은 약 17초).
 
-### 6-3. 엔진 단위 테스트 (DB 불필요, 가장 빠른 검증)
-```powershell
+### 6-3. 단위 테스트 (DB 불필요, 가장 빠른 검증)
+```bash
+cd backend
+python -m pytest -q
+```
+- 기대 결과: `34 passed` (약 1~5초).
+- `backend/pytest.ini`가 `backend/`를 import 경로에 넣어 주므로 `pytest -q`만 써도 됩니다.
+  (이 파일이 없을 때는 `pytest -q`가 `ModuleNotFoundError: No module named 'app'`으로
+  4개 파일 모두 실패했습니다.)
+
+| 파일 | 개수 | 검증 내용 |
+|---|---|---|
+| `tests/test_matching.py` | 5 | 기존 그리디 엔진(`app/matching.py`) |
+| `tests/test_match_guards.py` | 3 | 매칭 라우터의 중복 배정 방어 |
+| `tests/test_engine_v2.py` | 9 | 엔진 계약: 점수 범위, J 패널티, 전원 배정, 시드 재현, 사전 점검, 주관식 해석 |
+| `tests/test_engine_port.py` | 17 | AI 엔진 v1.1: 팀 수 규칙, 태그 별칭, 역할 배정, 빠른 채점기 = 기준 계산, 하드 조건 준수, 완화안(fix), evaluate, 학과·학년 미입력 |
+
+### 6-4. 엔진 시연 (DB 불필요)
+```bash
 cd backend
-pytest -q
+python scripts/engine_demo.py            # 합성 수강생 45명, 교수 메모 포함
+python scripts/engine_demo.py --n 90     # 90명
+python scripts/engine_demo.py --no-note  # 교수 메모 없이
 ```
+- 사전 점검 → 편성 → 검산(4학년 분산, 발표·개발 희망자, 같은 학과 수) → 팀별 요약·역할 →
+  조건 충족 표 → 대안 → evaluate(학생 1명 맞바꾼 뒤 재채점) 순서로 출력합니다.
+- 45명 기준 기대값: 약 1초, 평균 86.9 · 최저 83.8, 하드 위반 0.
 
 ---
 
@@ -248,10 +292,12 @@ pytest -q
 5. **테스트가 엔진에 치우침**
    - `tests/`는 매칭 엔진 로직만 검증합니다. 라우터(인증/권한/수업 흐름)
      레벨 테스트가 없어, API 리그레션을 못 잡습니다.
-   - 게다가 v2 테스트는 아직 `pytest`로 **실행 검증을 하지 않았습니다.**
+   - (2026-10-10 갱신) 엔진 테스트는 `python -m pytest -q`로 34개 모두 통과를
+     확인했습니다. 라우터 레벨 테스트가 없는 점은 그대로입니다.
 
 6. **운영 설정 미비**
-   - `.gitignore`, 배포 설정(Docker 등), 운영용 비밀키 관리가 없습니다.
+   - 배포 설정(Docker 등), 운영용 비밀키 관리가 없습니다.
+     (`.gitignore`는 저장소 루트에, `.env.example`은 `backend/`에 추가됨 — 2026-10-10)
    - `config.py`의 `jwt_secret` 기본값이 placeholder라 운영 전 교체 필수.
 
 7. **CORS 출처가 예시값**
diff --git a/backend/pytest.ini b/backend/pytest.ini
new file mode 100644
index 0000000..94e3e4b
--- /dev/null
+++ b/backend/pytest.ini
@@ -0,0 +1,4 @@
+[pytest]
+# backend/를 import 경로에 넣어 `pytest`만으로도 `from app ...`이 동작하게 함
+pythonpath = .
+testpaths = tests
-- 
2.43.0


From 60bf9408d6607090ddaabf568da75f7792f6e9bb Mon Sep 17 00:00:00 2001
From: =?UTF-8?q?=EC=A0=95=ED=9D=AC=EC=B0=AC?= <dc07080@hanyang.ac.kr>
Date: Sat, 10 Oct 2026 13:24:10 +0900
Subject: [PATCH 2/2] =?UTF-8?q?backend/README.md=20=EC=B2=AB=EB=A8=B8?=
 =?UTF-8?q?=EB=A6=AC=20=EC=A0=95=EB=A6=AC?=
MIME-Version: 1.0
Content-Type: text/plain; charset=UTF-8
Content-Transfer-Encoding: 8bit

한 문단으로 뭉쳐 보이던 소개·스택·폴더 구조·설치·프론트 연동·데이터 모델·기존 엔진 설명을
제목, 표, 코드 블록으로 나눠 다시 씀. 내용은 그대로 두고,
- 폴더 구조를 실제 파일에 맞게 갱신 (engine/, matching_v2.py, scripts/, tests/, pytest.ini 등)
- 설치 명령에 macOS/Linux 추가
- 프론트 연동 표에 precheck·match-v2 행 추가
- 번역 안내 문구 "(위 코드 블록의 … 번역한 것입니다.)" 삭제

Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01M7wfP91feLoQnFqvNdTruV
---
 backend/README.md | 122 ++++++++++++++++++++++++++++++++++++++--------
 1 file changed, 102 insertions(+), 20 deletions(-)

diff --git a/backend/README.md b/backend/README.md
index f13e063..5e1e689 100644
--- a/backend/README.md
+++ b/backend/README.md
@@ -1,22 +1,56 @@
-index.html 프론트엔드를 위한 Python (FastAPI) + PostgreSQL 백엔드입니다. 현재 프론트엔드가 state에 보관하고 있는 모든 데이터를 영구적으로 저장하며, 단 하나의 핵심 비즈니스 로직인 팀 매칭을 전담합니다.   스택 (Stack)FastAPI — HTTP API 및 유효성 검사   SQLAlchemy 2.0 — ORM (app/models.py, db/schema.sql과 동일하게 구성)   PostgreSQL — 데이터 저장소   JWT (python-jose) + bcrypt (passlib) — 인증   매칭 엔진은 순수 파이썬(pure Python)으로 작성되었으며(app/matching.py), DB와 독립적으로 동작하고 단위 테스트(unit-test)를 거쳤습니다.   폴더 구조 (Layout)Plaintextbackend/
-  db/schema.sql          # PostgreSQL DDL + 기본 제공 프리셋(preset) 시드 데이터
-  app/
-    config.py            # 환경변수 설정
-    database.py          # 엔진 + 세션 + get_db 의존성
-    models.py            # ORM 모델 (schema.sql과 1:1 대응)
-    security.py          # 비밀번호 해싱, JWT, 인증 의존성
-    schemas.py           # Pydantic 요청/응답 모델
-    matching.py          # 팀 매칭 엔진 (순수 함수)
-    main.py              # 앱 진입점 + CORS + 라우터 연결
-    routers/
-      auth.py            # 회원가입 / 로그인 / 내 정보 확인
-      classes.py         # 교수용 수업 CRUD + 명단; 학생 참여 코드(join-by-code) 입력
-      surveys.py         # 수업별 설문 제출 / 조회
-      matching.py        # 매칭 실행, 프리셋 목록 조회
-  tests/test_matching.py # 엔진 단위 테스트
-  requirements.txt
-  .env.example
-(위 코드 블록의 구조 및 주석 내용은 원문 코드를 번역한 것입니다.)   설정 및 실행 (Setup)PowerShell# backend/ 폴더 안에서 실행
+# TeamMatch 백엔드
+
+`index.html` 프론트엔드를 위한 Python(FastAPI) + PostgreSQL 백엔드입니다. 지금 프론트엔드가 `state`에 들고 있는 데이터를 DB에 영구 저장하고, 핵심 비즈니스 로직인 **팀 매칭**을 맡습니다.
+
+## 스택
+
+| 구성 | 역할 |
+|---|---|
+| FastAPI | HTTP API, 요청 유효성 검사 |
+| SQLAlchemy 2.0 | ORM (`app/models.py`, `db/schema.sql`과 같은 구조) |
+| PostgreSQL | 데이터 저장소 |
+| JWT(python-jose) + bcrypt(passlib) | 인증 |
+
+매칭 엔진(`app/matching.py`, `app/engine/`)은 순수 파이썬이라 DB 없이 동작하고 단위 테스트로 검증합니다.
+
+## 폴더 구조
+
+```text
+backend/
+├── app/
+│   ├── config.py            # 환경변수 설정
+│   ├── database.py          # DB 엔진 + 세션 + get_db 의존성
+│   ├── models.py            # ORM 모델 (schema.sql과 1:1 대응)
+│   ├── security.py          # 비밀번호 해싱, JWT, 인증 의존성
+│   ├── schemas.py           # Pydantic 요청/응답 모델
+│   ├── matching.py          # 기존 그리디 매칭 엔진 (순수 함수)
+│   ├── main.py              # 앱 진입점 + CORS + 라우터 연결
+│   ├── engine/              # TeamFit AI 엔진 v1.1 (설명: app/engine/README.md)
+│   └── routers/
+│       ├── auth.py          # 회원가입 / 로그인 / 내 정보
+│       ├── classes.py       # 교수용 수업 CRUD + 명단, 학생 참여 코드 입력
+│       ├── surveys.py       # 수업별 설문 제출 / 조회
+│       ├── matching.py      # 기존 매칭 실행, 프리셋 목록
+│       └── matching_v2.py   # AI 엔진 사전 점검(precheck) / 매칭(match-v2)
+├── db/
+│   ├── schema.sql           # PostgreSQL DDL + 기본 프리셋 시드 데이터
+│   └── schema.mysql.sql     # 같은 스키마의 MySQL 버전
+├── scripts/engine_demo.py   # 합성 데이터로 AI 엔진 시연 (DB 불필요)
+├── tests/                   # 단위 테스트 34개 (TESTING.md 6-3)
+├── requirements.txt
+├── pytest.ini
+├── .env.example
+├── TESTING.md               # 처음부터 끝까지 테스트하는 방법
+└── TODO.txt                 # 남은 작업
+```
+
+## 설정 및 실행
+
+`backend/` 폴더 안에서 실행합니다.
+
+Windows (PowerShell):
+
+```powershell
 python -m venv .venv
 .\.venv\Scripts\Activate.ps1
 pip install -r requirements.txt
@@ -26,10 +60,58 @@ psql -U postgres -c "CREATE DATABASE teammatch;"
 psql -U postgres -d teammatch -f db/schema.sql
 
 copy .env.example .env   # 복사 후 값 수정
+uvicorn app.main:app --reload
+```
+
+macOS / Linux:
+
+```bash
+python3 -m venv .venv
+source .venv/bin/activate
+pip install -r requirements.txt
+
+psql -U postgres -c "CREATE DATABASE teammatch;"
+psql -U postgres -d teammatch -f db/schema.sql
 
+cp .env.example .env     # 복사 후 값 수정
 uvicorn app.main:app --reload
-(위 코드 블록의 명령어 주석은 원문 코드를 번역한 것입니다.)   대화형 API 문서: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)   테스트 실행: backend/ 폴더에서 python -m pytest -q (34개, 자세한 방법은 TESTING.md).   프론트엔드 연동 매핑 (How it maps to the frontend)프론트엔드 (index.html)백엔드authRole 학생/교수, 로그인/회원가입   POST /api/auth/signup, POST /api/auth/login, GET /api/auth/me   state.prof.classes[], 참여 코드 c.join   POST /api/classes, GET /api/classes, POST /api/classes/join   ROSTER (학생 명단) + 제출 상태   GET /api/classes/{id}/roster   state.user 역할 / 스킬 / 성향 / 일정   PUT/GET /api/classes/{id}/survey   setup.weights + hard + 프리셋   POST /api/classes/{id}/match, GET /api/presets   "AI 팀매칭" 실행 + state.prof.result   매칭 결과는 match_runs + teams + team_members에 저장됨   데이터 모델 참고 사항하나의 users 테이블을 사용하며 role(역할)로 구분합니다. 학생 전용 및 교수 전용 컬럼은 Null 값을 허용(nullable)합니다.   설문 데이터(Survey signals)는 수강 내역(enrollment)을 기준으로 survey_responses (+ survey_skills, survey_availability)에 저장되므로, 학생은 수업마다 다르게 답변할 수 있습니다.   가용 시간(Availability)은 프론트엔드의 "day-block" 그리드를 선택된 슬롯당 한 줄씩 데이터로 저장합니다. day 0=월 .. 6=일이며, block은 프론트엔드의 BLOCKS[] 인덱스와 일치합니다.   owner_id IS NULL (is_builtin=true)인 프리셋(Presets)은 schema.sql 하단에 시드 데이터로 제공되는 공유 기본값(프론트엔드의 ps0..ps4)입니다.   매칭 엔진 (app/matching.py)   form_teams(candidates, team_size, weights, hard) 함수는 그리디(greedy) 밸런싱 휴리스틱(탐색 기법)을 사용합니다:   팀 수 = ceil(n / team_size)로 계산합니다.   가장 희귀한 역할을 먼저 각 팀에 하나씩 배정한 후(seed), 남은 학생들은 추가 시 팀을 가장 크게 개선할 수 있는 곳에 그리디 방식으로 배정합니다(가능한 한 하드 제약 조건을 준수함).   각 팀은 교수가 설정한 1~5점 가중치를 바탕으로 4가지 축을 혼합하여 0~100점의 조화도 점수(fit score)를 받습니다:   major — 전공 다양성 (또는 same_major 하드 제약 조건이 켜져 있을 경우 동일성)   role — 서로 다른 역할의 충족도   time — 가용한 일정 슬롯의 겹침 정도   trait — 팀원 간 작업 스타일(성향)의 유사도   이것은 정확한 최적화 알고리즘이 아니라 휴리스틱 탐색으로, 설명 가능하고 좋은 팀을 빠르게 구성해 줍니다. 만약 나중에 수학적으로 완벽한 최적의 배정이 필요하다면, API나 DB 계층을 건드릴 필요 없이 이 모듈 하나만 ILP(정수계획법)/제약 조건 솔버(solver)로 교체하면 됩니다. 
+```
 
+- 대화형 API 문서: <http://127.0.0.1:8000/docs>
+- 테스트: `python -m pytest -q` → 34개 통과 (자세한 방법은 [TESTING.md](TESTING.md))
+
+## 프론트엔드 연동 매핑
+
+| 프론트엔드 (`index.html`) | 백엔드 |
+|---|---|
+| `authRole` 학생/교수, 로그인/회원가입 | `POST /api/auth/signup`, `POST /api/auth/login`, `GET /api/auth/me` |
+| `state.prof.classes[]`, 참여 코드 `c.join` | `POST /api/classes`, `GET /api/classes`, `POST /api/classes/join` |
+| `ROSTER`(학생 명단) + 제출 상태 | `GET /api/classes/{id}/roster` |
+| `state.user` 역할 / 스킬 / 성향 / 일정 | `PUT`·`GET /api/classes/{id}/survey` |
+| `setup.weights` + `hard` + 프리셋 | `POST /api/classes/{id}/match`, `GET /api/presets` |
+| "AI 팀매칭" 실행 + `state.prof.result` | 매칭 결과는 `match_runs` + `teams` + `team_members`에 저장 |
+| 조건 점검 + AI 엔진 매칭 | `POST /api/classes/{id}/precheck`, `POST /api/classes/{id}/match-v2` |
+
+## 데이터 모델 참고 사항
+
+- `users` 테이블 하나를 `role`(역할)로 구분합니다. 학생 전용·교수 전용 컬럼은 Null을 허용합니다.
+- 설문 데이터는 수강 내역(enrollment) 기준으로 `survey_responses`(+ `survey_skills`, `survey_availability`)에 저장되므로, 학생은 수업마다 다르게 답할 수 있습니다.
+- 가용 시간은 프론트엔드의 "day-block" 그리드에서 선택된 칸마다 한 줄씩 저장합니다. `day` 0=월 … 6=일, `block`은 프론트엔드 `BLOCKS[]` 인덱스(0 = 09–10시 … 11 = 20–21시)와 같습니다.
+- `owner_id IS NULL`(`is_builtin=true`)인 프리셋은 `schema.sql` 하단의 시드 데이터로, 모두가 쓰는 기본값(프론트엔드 `ps0`…`ps4`)입니다.
+
+## 기존 매칭 엔진 (`app/matching.py`)
+
+`form_teams(candidates, team_size, weights, hard)`는 그리디 밸런싱 휴리스틱입니다. `POST /api/classes/{id}/match`가 사용합니다.
+
+1. 팀 수 = `ceil(n / team_size)`
+2. 가장 희귀한 역할을 먼저 팀마다 하나씩 배정(seed)한 뒤, 남은 학생은 팀 점수를 가장 크게 올리는 곳에 배정합니다 (하드 조건은 가능한 한 지킴).
+3. 각 팀은 교수가 정한 1~5 가중치로 4개 축을 섞어 0~100점의 조화도 점수(fit score)를 받습니다.
+    - `major` — 전공 다양성 (`same_major` 하드 조건이 켜지면 같은 전공일수록 높음)
+    - `role` — 서로 다른 역할의 충족도
+    - `time` — 가용 시간 겹침
+    - `trait` — 팀원 간 작업 성향의 유사도
+
+정확한 최적화가 아니라 설명 가능하고 빠른 휴리스틱입니다. 더 정교한 최적화는 `app/engine/`의 AI 엔진(`match-v2`)이 맡습니다 — 아래 "TeamFit AI 매칭 엔진" 절과 [app/engine/README.md](app/engine/README.md) 참고.
 
 ---
 
-- 
2.43.0
