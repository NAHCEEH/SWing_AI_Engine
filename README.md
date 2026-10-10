# SWing_AI_Engine
Sw Capstone Design_AI Engine_Modeling

# SWing AI Engine

팀 SWing의 **TeamFit 팀 편성 엔진과 FastAPI 백엔드**를 담은 저장소입니다. DS 창업캡스톤디자인 프로젝트로, 학생 설문과 교수의 편성 조건을 반영해 수업별 팀을 구성합니다.

TeamFit은 학생의 희망 역할, 가용 시간, 협업 성향과 전공 정보를 함께 평가합니다. 팀별 조화도를 계산하고 학생 배치를 개선해, 전체 팀의 평균 점수와 가장 낮은 팀의 점수를 함께 높이는 편성을 탐색합니다.

엔진은 팀 점수 함수, 조합 최적화, 규칙 기반 교수 메모 해석기로 구성됩니다.

## 주요 기능

- **학생·수업 관리**: 학생·교수 인증, 수업 생성 및 참여, 수강생 조회
- **설문 수집**: 수업별 희망 역할, 스킬, 작업 성향, 가용 시간 저장
- **조건 사전 점검**: 팀 편성 전 조건 충족 가능 여부 확인 및 완화안 안내
- **자동 팀 편성**: 전공, 역할, 시간, 성향과 교수의 제약 조건을 반영한 매칭
- **결과 분석**: 팀 점수, 역할 배정, 조건 충족 여부, 팀별 설명 및 대안 편성 계산
- **엔진 시연**: 데이터베이스 없이 합성 수강생 데이터로 편성 과정 확인

결과 분석 기능의 API 노출 범위와 요청·응답 형식은 백엔드 API 문서에서 확인할 수 있습니다.

## 기술 구성

| 영역 | 기술 및 방식 |
| --- | --- |
| 백엔드 API | Python, FastAPI, Pydantic |
| 데이터베이스 | PostgreSQL, SQLAlchemy 2.0, psycopg |
| 인증 | JWT(python-jose), 비밀번호 해싱(passlib/bcrypt) |
| 팀 편성 | 그리디 초기 배치, 국소 탐색, 담금질 기법, 다중 시작 |
| 교수 메모 해석 | 정규식 기반 규칙 처리 |
| 테스트 | pytest |
| 프론트엔드 | 정적 HTML 프로토타입 |

## 빠른 시작

Python이 설치된 환경에서 저장소를 내려받습니다.

```bash
git clone https://github.com/NAHCEEH/SWing_AI_Engine.git
cd SWing_AI_Engine/backend
```

### macOS / Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

### Windows / PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

### 테스트 및 시연

가상환경을 활성화한 상태에서 `backend/` 폴더 안에서 실행합니다. 엔진 단위 테스트와 시연에는 실행 중인 데이터베이스와 `.env`가 필요하지 않습니다.

```bash
python -m pytest -q
python scripts/engine_demo.py
```

기본 시연은 합성 수강생 45명과 교수 메모를 사용합니다. 사전 점검, 편성, 조건 검산, 팀별 요약과 역할 배정, 재평가 결과를 출력합니다.

```bash
python scripts/engine_demo.py --n 90
python scripts/engine_demo.py --no-note
```

`--n`은 수강생 수를 지정하고, `--no-note`는 교수 메모 없이 실행합니다.

백엔드 서버를 실행하려면 PostgreSQL과 환경변수 설정이 필요합니다. 자세한 절차는 [백엔드 README](backend/README.md)를 참고하세요.

## 저장소 구조

```text
SWing_AI_Engine/
├── backend/
│   ├── app/
│   │   ├── engine/          # TeamFit 매칭 엔진
│   │   ├── routers/         # 인증, 수업, 설문, 매칭 API
│   │   ├── matching.py      # 기존 그리디 매칭 엔진
│   │   └── main.py          # FastAPI 앱 진입점
│   ├── db/                 # DB 스키마 및 기본 프리셋
│   ├── scripts/            # 엔진 시연
│   ├── tests/              # 단위 테스트
│   ├── requirements.txt
│   ├── pytest.ini
│   ├── .env.example
│   ├── README.md
│   └── TESTING.md
├── docs/
│   └── api_examples/        # API 요청·응답 예시
├── index.html              # 프론트엔드 프로토타입
└── README.md
```

## TeamFit 엔진 동작

```mermaid
flowchart LR
    A[학생 설문과 교수 조건] --> B[메모 해석 및 사전 점검]
    B --> C[초기 팀 배치]
    C --> D[국소 탐색 및 담금질]
    D --> E[여러 시작점의 결과 비교]
    E --> F[팀 편성 및 결과 설명]
```

1. **메모 해석 및 사전 점검**: 교수 메모를 조건으로 변환하고, 학과별 인원 제한과 역할별 최소 인원 등 요청을 충족할 수 있는지 확인합니다.
2. **초기 배치**: 같은 팀에 고정할 학생을 묶고, 희소한 조건을 가진 학생부터 배정합니다.
3. **배치 개선**: 팀 간 학생 교환과 이동으로 점수를 개선합니다. 담금질 탐색은 점수가 일시적으로 낮아지는 교환도 확률적으로 허용해 다른 배치를 탐색합니다.
4. **결과 선택**: 여러 시작점에서 탐색한 결과를 비교해 목적함수가 가장 높은 편성을 선택합니다.
5. **결과 설명**: 팀 구성, 역할 배정, 점수, 조건 충족 여부와 대안을 계산합니다.

### 팀 점수와 최적화 목표

팀 점수는 다음 네 항목에 교수의 가중치를 적용해 0~100점으로 계산합니다.

| 평가 항목 | 내용 |
| --- | --- |
| 전공 | 팀 내 전공 다양성 |
| 역할 | 서로 다른 역할의 배정과 희망 순위 충족도 |
| 시간 | 팀원 모두가 참여할 수 있는 공통 시간 |
| 성향 | 작업 페이스, 의사결정, 소통 빈도 등 협업 성향 |

```text
팀 점수 = 100 × Σ(가중치 × 하위 점수) / Σ가중치
목적함수 J = 평균 팀 점수 + 최저 팀 점수 − 1000 × 하드 조건 위반 수
```

하드 조건은 같은 학과의 최대 인원, 특정 학생의 분리·고정, 팀별 역할 최소 인원, 학년 분산 등을 포함합니다.

TeamFit 엔진의 내부 구현은 v1.1이며, API는 `/api/classes/{class_id}/precheck`와 `/api/classes/{class_id}/match-v2` 경로로 제공합니다. 기존 그리디 엔진은 `/api/classes/{class_id}/match`에서 사용합니다.

알고리즘별 구현과 세부 계산식은 [엔진 README](backend/app/engine/README.md)를 참고하세요.

## 검증 범위

단위 테스트는 기존 매칭 엔진, 중복 배정 방어, 점수 범위, 전원 배정, 시드 재현성, 사전 점검, 태그 해석과 제약 조건 등을 검증합니다. 실행 방법과 파일별 검증 내용은 [테스트 안내](backend/TESTING.md)에 정리되어 있습니다.

엔진은 휴리스틱 탐색을 사용하므로 최적해를 보장하지 않습니다. 검증은 합성 데이터를 기준으로 하며, 실제 설문 데이터에 대한 검증과 인증·권한·수업 흐름 전체에 대한 API 통합 테스트는 추가로 필요합니다.

## 관련 문서

| 문서 | 내용 |
| --- | --- |
| [백엔드 README](backend/README.md) | 설치·실행, 환경변수, 주요 API, 데이터 모델 |
| [테스트 안내](backend/TESTING.md) | 단계별 API 검증, 단위 테스트, 엔진 시연 |
| [엔진 README](backend/app/engine/README.md) | 엔진 구성, 점수 계산, 최적화 및 제약 조건 |
| [API 예시](docs/api_examples/) | 요청·응답 JSON 및 오류 응답 예시 |
