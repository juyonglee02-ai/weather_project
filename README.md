🏗️ 시스템 아키텍처 요약 (GitHub README용)
1. 개요
본 프로젝트는 에지 컴퓨팅(ESP32)과 클라우드 서버(RPi 5 + Spring Boot)를 결합하여 실시간 환경 모니터링 및 AI(LLM) 분석 서비스를 제공하는 통합 IoT 시스템입니다.

2. 기술 스택 (Tech Stack)
Hardware: ESP32 NodeMCU, AM 2302(온습도) ,GP2Y1014(미세먼지),

Infrastructure: Raspberry Pi 5 (Linux), PostgreSQL

Backend: Java 17, Spring Boot, JPA, RestClient

Frontend: React, Chart.js/Recharts, Tailwind CSS

AI: LM Studio (Local LLM Server), OpenAI-compatible API

External API: 기상청 단기예보, 에어코리아 대기질 공공데이터

3. 데이터 흐름 (Data Flow)
Sensing: ESP32가 센서 데이터를 수집하여 JSON 형식으로 변환.

Transmission: WiFi를 통해 Spring Boot API 서버로 POST 요청.

Storage: PostgreSQL에 시계열 데이터 저장 및 이력 관리.

Enrichment: Spring Boot에서 외부 공공 API를 호출하여 지역 기상 정보와 로컬 센싱 데이터 결합.

Intelligence: 수집된 데이터를 프롬프트로 구성하여 로컬 LLM(LM Studio)에 전달, 사용자 맞춤형 분석 결과 생성.

Visualization: React 대시보드에서 실시간 데이터 및 AI 챗봇 인터페이스 제공.

💡 PPT 발표 포인트 (Tips)
라즈베리 파이 5 활용 강조: 단순한 보드가 아닌 '리눅스 가상 서버'로서 백엔드와 DB를 안정적으로 호스팅하고 있다는 점을 강조하세요.

AI 모델 연동: 외부 유료 API가 아닌 LM Studio를 이용한 로컬 LLM 연동을 통해 데이터 보안과 비용 효율성을 챙겼다는 점이 큰 장점입니다.

데이터 융합: 내 집 안의 센서 데이터와 기상청의 공공 데이터를 비교 분석하는 로직(Data Cross-Validation)을 언급하면 프로젝트의 완성도가 높아 보입니다.

✔ 실시간 센서 데이터 수집 성공
✔ PostgreSQL 저장 성공
✔ 기상청 API 연동 성공
✔ 에어코리아 API 연동 성공
✔ Spring Boot REST API 구축 성공
✔ React Dashboard 구현 성공
✔ AI 예측 모델 구현 성공
✔ LM Studio 기반 챗봇 구현 성공
✔ Raspberry Pi 5 Linux 서버 구축 성공
