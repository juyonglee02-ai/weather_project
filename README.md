# 🌤️ 내 위치 기반 날씨·미세먼지 예측 시스템

> IoT 센서와 AI를 결합한 초개인화 환경 모니터링 플랫폼

[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.3.5-6DB33F?style=flat&logo=springboot)](https://spring.io/)
[![React](https://img.shields.io/badge/React-Vite-61DAFB?style=flat&logo=react)](https://react.dev/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?style=flat&logo=postgresql)](https://www.postgresql.org/)
[![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=flat&logo=python)](https://python.org/)
[![LM Studio](https://img.shields.io/badge/LM_Studio-Qwen3--14b-FF6B35?style=flat)](https://lmstudio.ai/)

---

## 📌 프로젝트 동기

대한민국의 날씨는 사계절과 지형 특성으로 지역별 편차가 크고, 기상청 예보는 광역 단위라 **내 동네 날씨와 다를 수 있습니다.**  
미세먼지 역시 중국발 황사·공장 배출 등 다양한 변수가 복합 작용해 **공식 측정소와 내 위치 사이에 오차가 발생**합니다.

외부에서 일하는 사람, 호흡기 질환자 등 날씨·대기질 변화에 민감한 사람들이 **지금 내가 있는 곳의 데이터로 즉각 판단**할 수 있도록 이 시스템을 구축했습니다.

---

## 🏗️ 시스템 아키텍처

```
[ESP32 센서] ──────────────────────────┐
  온습도 · 미세먼지 직접 측정            │
                                        ▼
[기상청 API] ──→ [Spring Boot] ──→ [PostgreSQL]
[에어코리아 API]  @Scheduled 3h          │
                  자동 수집              │
                                        ├──→ [Python 예측서버 :5001]
                                        │    RandomForest 7일 예측
                                        │
                                        └──→ [Python LLM서버 :5002]
                                             LM Studio · Qwen3-14b
                                                  │
                              [React 대시보드] ←──┘
                              날씨 · 대기질 · AI예측 · 챗봇
```

---

## ⚙️ 기술 스택

| 레이어 | 기술 |
|--------|------|
| **하드웨어** | ESP32, 온습도 센서, 미세먼지 센서, Raspberry Pi 5 |
| **백엔드** | Spring Boot 3.3.5, Spring Data JPA, @Scheduled |
| **데이터베이스** | PostgreSQL 16 (Linux VM) |
| **AI 예측** | Python 3.12, Flask, scikit-learn (RandomForest), pandas |
| **LLM 챗봇** | LM Studio, Qwen3-14b, OpenAI 호환 API |
| **프론트엔드** | React 18, Vite, Custom Hook |
| **외부 API** | 기상청 단기예보 API, 에어코리아 대기질 API |

---

## 🗄️ 데이터베이스 구조

```sql
weather_data     -- 기상청 API 날씨 (temperature, humidity, rain_probability, weather_state)
airquality_data  -- 에어코리아 대기질 (pm10, pm25, o3, no2, co, so2, air_index)
sensor_data      -- ESP32 직접 측정 (temperature, humidity, dust)
ai_predictions   -- AI 예측 결과 (forecast_date, predicted_temp, confidence)
```

---

## 🤖 AI 예측 모델

1. `sensor_data` + `weather_data` 를 1시간 단위로 JOIN → **284건 매칭**
2. 센서값 - API값 = **보정 델타(Δ)** 계산
   - 온도 Δ: −6.25°C / 습도 Δ: +12.19% / 먼지 Δ: +18.87
3. 보정된 피처로 **RandomForest** 학습 (기온 회귀 + 날씨상태 분류)
4. **날씨 수집 시 자동 재학습** (`@Scheduled` → `/train` 자동 호출)

```bash
# 모델 학습
curl -X POST http://localhost:5001/train

# 7일 예측
curl http://localhost:5001/predict

# 센서 vs API 비교 통계
curl http://localhost:5001/compare
```

---

## 💬 LLM 날씨 챗봇

- **LM Studio + Qwen3-14b** 로컬 모델 (비용 없음, 데이터 외부 유출 없음)
- PostgreSQL에서 실시간 날씨·대기질·센서·예측 데이터 조회
- 시스템 프롬프트로 주입 → 데이터 기반 자연어 답변 생성
- Qwen3 `<think>` 블록 정규식 제거로 깔끔한 답변 출력

---

## 🚀 실행 방법

### 1. PostgreSQL 설정
```bash
sudo -u postgres psql -d sensor -c "
CREATE TABLE IF NOT EXISTS ai_predictions (
    id BIGSERIAL PRIMARY KEY,
    forecast_date DATE,
    predicted_temp FLOAT,
    predicted_rain_prob INTEGER,
    confidence FLOAT,
    model_version VARCHAR(30),
    created_at TIMESTAMP DEFAULT NOW()
);"
```

### 2. Python 서버 실행
```bash
cd ai-server
python3 -m venv venv && source venv/bin/activate
pip install flask psycopg2-binary pandas scikit-learn openai python-dotenv joblib

# .env 설정
echo "DB_HOST=localhost
DB_NAME=sensor
DB_USER=postgres
DB_PASS=your_password
LM_STUDIO_URL=http://YOUR_WINDOWS_IP:1234" > .env

python predict_server.py   # 포트 5001
python llm_chat_server.py  # 포트 5002
```

### 3. Spring Boot 실행
```bash
cd sensor-server
# application.properties에 DB 정보 및 기상청 API 키 설정
./mvnw spring-boot:run  # 포트 8081
```

### 4. React 실행 (Windows)
```powershell
cd weather-dashboard
npm install && npm run dev  # 포트 5173
```

---

## 📡 API 엔드포인트

| 메서드 | 경로 | 설명 |
|--------|------|------|
| GET | `/api/weather/today` | 최신 날씨 1건 |
| GET | `/api/weather/weekly` | 최근 7일 날씨 |
| GET | `/api/air/today` | 최신 대기질 |
| GET | `/api/predict/weather` | AI 예측 결과 |
| POST | `/api/predict/run` | 예측 갱신 트리거 |
| POST | `/api/chat` | LLM 날씨 챗봇 |

---

## 🔧 트러블슈팅

| 문제 | 해결 |
|------|------|
| PostgreSQL Peer 인증 실패 | `pg_hba.conf` → `peer` to `trust` |
| Python externally-managed | `python3 -m venv` 가상환경 구성 |
| 테이블 권한 denied | `GRANT ALL ON TABLE ... TO sensoruser` |
| 다른 IP 간 LLM 연동 | LM Studio "Serve on Local Network" 활성화 + 방화벽 포트 개방 |
| AI 예측값 동일 | 학습 데이터 부족 → 기상청 과거 API 보강 필요 |

---

## 📁 프로젝트 구조

```
weather-project/
├── sensor-server/          # Spring Boot 백엔드
│   └── src/main/java/com/sensor/
│       ├── weather/        # 날씨 API
│       ├── air/            # 대기질 API
│       ├── predict/        # AI 예측 연동
│       ├── chat/           # LLM 챗봇 프록시
│       └── config/         # CORS 설정
├── ai-server/              # Python AI 서버
│   ├── predict_server.py   # RandomForest 예측 (포트 5001)
│   └── llm_chat_server.py  # LLM 챗봇 (포트 5002)
└── weather-dashboard/      # React 프론트엔드
    └── src/
        ├── App.jsx
        ├── hooks/useWeatherData.js
        └── components/WeatherChat.jsx
```

---

> 이 시스템은 외부에서 일하는 모든 사람을 위한 날씨 도우미입니다 ☀️
