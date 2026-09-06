# Crypto AI Assistant

하이퍼리퀴드(Hyperliquid) 거래소의 암호화폐 시계열 데이터를 분석하고,
그 데이터를 기반으로 대화하는 AI 챗봇 서비스입니다.
"내 상황을 아는 AI 비서"를 목표로, 데이터 분석 → 저장 → AI 대화까지 이어지는 흐름을 구현했습니다.

## 배포 URL

| 항목 | 링크 |
|---|---|
| 프론트엔드 (실제 서비스) | https://jh-3-2-frontend.vercel.app |
| 백엔드 API | https://jh-cs-3-2.onrender.com |
| Swagger API 문서 | https://jh-cs-3-2.onrender.com/docs |

## 저장소

| 구분 | 저장소 |
|---|---|
| 백엔드 (FastAPI) | https://github.com/EC-newsbot/JH-CS-3-2 |
| 프론트엔드 (HTML/CSS/JS) | https://github.com/EC-newsbot/JH-3-2-frontend |

## 기술 스택

- **백엔드**: FastAPI, Python 3
- **프론트엔드**: HTML, CSS, JavaScript (바닐라, 프레임워크 미사용), Chart.js
- **데이터베이스**: Firebase Firestore
- **AI**: OpenAI GPT API (gpt-4o-mini), Function Calling
- **데이터 소스**: Hyperliquid API (BTC, ETH, HYPE, XRP 등)
- **배포**: Render (백엔드), Vercel (프론트엔드)

## 주요 기능

1. **데이터 기반 AI 채팅**: 사용자의 질문에 따라 GPT가 필요한 코인 데이터를 스스로 조회(Function Calling)하여 맞춤 답변 제공
2. **데이터 관리 (CRUD)**: 코인별 가격 데이터 추가/조회/수정/삭제
3. **대화 기록 저장 및 불러오기**: 챗봇과 나눈 대화를 자동 저장, 목록 조회 및 상세 불러오기
4. **요약 정보 + 그래프**: 코인별 통계(평균/최대/최소/트렌드)와 가격 추이 그래프, 기간 필터링 지원
5. **(보너스) 변동성 상위 10개 코인**: 거래대금 상위 코인 중 최근 30일 변동률(절대값) 기준 상위 10개를 매일 배치로 계산
6. **(보너스) Function Calling**: GPT가 사용자 질문을 분석해 `get_coin_summary`, `get_top_movers` 중 필요한 함수를 스스로 호출

## 데이터 범위 제한 사항

- 본 서비스는 Hyperliquid 거래소의 최근 약 180일치 일봉 데이터를 기반으로 합니다. 그 이전 기간에 대한 질문은 데이터 부재로 응답이 제한됩니다.
- "변동성 상위 코인" 기능은 조회 시점 기준 최근 30일 데이터만 제공하며, 과거 특정 시점 조회는 지원하지 않습니다.
- 변동성 상위 코인은 거래대금 상위 50개 코인 중에서만 산출됩니다.

## 로컬 실행 방법

### 백엔드

```bash
git clone https://github.com/EC-newsbot/JH-CS-3-2.git
cd JH-CS-3-2
python -m venv venv
venv\Scripts\Activate.ps1   # Windows
pip install -r requirements.txt
# .env 파일 생성 후 아래 "환경 변수" 참고하여 값 설정
# firebase-key.json 파일을 프로젝트 최상위에 위치
uvicorn app.main:app --reload
```

브라우저에서 `http://127.0.0.1:8000/docs` 접속하여 Swagger UI 확인 가능합니다.

### 프론트엔드

```bash
git clone https://github.com/EC-newsbot/JH-3-2-frontend.git
cd JH-3-2-frontend
```

`script.js` 상단의 `API_BASE_URL`을 로컬 백엔드 주소(`http://127.0.0.1:8000`) 또는 배포된 백엔드 주소로 설정한 뒤, `index.html`을 브라우저로 열면 됩니다.

## 환경 변수 (백엔드 .env)

| 변수명 | 설명 |
|---|---|
| `OPENAI_API_KEY` | OpenAI API 키 |
| `FIREBASE_SERVICE_ACCOUNT_PATH` | Firebase 서비스 계정 키 파일 경로 (로컬: `./firebase-key.json`, Render 배포 시: `/etc/secrets/firebase-key.json`) |

## 시스템 아키텍처

```
[사용자 브라우저]
      │ 접속
      ▼
[Vercel] 프론트엔드 (HTML/CSS/JS)
      │ API 호출
      ▼
[Render] 백엔드 (FastAPI)
      │ 데이터 읽기/쓰기        │ GPT 호출
      ▼                        ▼
[Firebase Firestore]      [OpenAI API]
(data, conversations,
 top_movers 컬렉션)
```

## API 엔드포인트 요약

| 메서드 | 경로 | 설명 |
|---|---|---|
| POST | `/api/data` | 데이터 추가 |
| GET | `/api/data` | 데이터 목록 조회 (coin 필터 가능) |
| PUT | `/api/data/{doc_id}` | 데이터 수정 |
| DELETE | `/api/data/{doc_id}` | 데이터 삭제 |
| GET | `/api/data/summary` | 코인별 요약 통계 (기간 필터 가능) |
| GET | `/api/data/top-movers` | 변동성 상위 10개 코인 조회 |
| POST | `/api/conversations` | 대화 저장 |
| GET | `/api/conversations` | 대화 목록 조회 (미리보기) |
| GET | `/api/conversations/{doc_id}` | 특정 대화 상세 조회 |
| DELETE | `/api/conversations/{doc_id}` | 대화 삭제 |
| POST | `/api/chat` | AI 챗봇 (Function Calling 기반) |

## 제출 스크린샷

### 데이터 요약이 보이는 채팅 화면
(스크린샷 추가 예정)

### 데이터 관리 화면
(스크린샷 추가 예정)

### 대화 기록 화면
(스크린샷 추가 예정)
