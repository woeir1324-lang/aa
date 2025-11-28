CODE5IVE | 경남 해역 전용 해양환경 모니터링 및 예측 플랫폼

🌟 프로젝트 개요

Marine Guard는 경남 해역의 해양 환경 문제를 해결하기 위해 개발된 실시간 모니터링 및 예측 플랫폼입니다. 국립수산과학원(RISA)의 공공 데이터를 기반으로 AI 분석을 결합하여, 기존의 사후 보고 중심 방식에서 벗어나 오염 악화 전 위험을 미리 예측하고 어민과 지자체가 즉각 대응할 수 있도록 지원합니다.

💡 서비스 목표

실시간 파악: 해양환경 변화를 실시간으로 자동 수집 및 파악.

위험 예측: AI 기반 분석을 통해 적조, 수질 악화 등 위험을 사전에 예측.

신속 대응: 어민, 지자체에 즉각적인 의사결정 정보를 제공하여 피해 최소화.

투명성 확보: 시민과 지역 단체에 투명한 환경 정보를 공개하여 참여 확대.

🛠️ 주요 기능 (Key Features)

아이콘

기능 명칭

상세 설명

📊

AI 기반 위험 분석 및 예측

수온, 탁도, 클로로필-a 농도 등 환경 변화 요인을 분석하여 오염 신호를 조기에 감지하며, 적조 및 쓰레기 이동 경로 등 향후 24~48시간의 위험 가능성을 예측 모델로 제공합니다.

🗺️

지도 기반 직관적 시각화

누구나 앱/웹에서 색상 등급(Good, Warning, Danger)만으로 지역별 실시간 위험도를 한눈에 확인할 수 있는 지도 UI를 제공합니다.

📡

공공 데이터 자동 수집

RISA 데이터를 API로 연동하여 최신 수질, 적조, 해양 정보를 실시간으로 자동 수집 및 갱신합니다.

🔔

사용자 맞춤형 알림

어민, 지자체, 일반 시민 등 사용자 그룹별로 필요한 정보와 위험 경보만 선별하여 받아볼 수 있는 맞춤 알림 기능을 제공합니다.

🚀 차별점 및 특장점

항목

기존 방식

Marine Guard (본 서비스)

데이터 갱신

정기 측정 (느림)

실시간 자동 수집

오염 파악

사후 보고 중심

발생 직전 예측

정보 제공

보고서 · 문서 중심

지도 · 시각화 중심 (직관성 극대화)

사용자 대상

공공기관 중심

어민 · 시민까지 확대

특장점 요약:

비용 효율적: 공공데이터 기반으로 시스템 운영 비용 절감.

빠르고 정확한 판단: AI 분석을 통해 신뢰성 높은 위험 예측 정보 제공.

지역 문제 직접 기여: 경남 지역에 특화된 서비스로 지역 사회 문제 해결에 직접적으로 기여.

💻 기술 스택 (Technology Stack)

본 프로젝트에 사용되거나 계획된 주요 기술 스택은 다음과 같습니다.

분류

기술 스택

설명

데이터 수집

RISA API

국립수산과학원 해양정보시스템 데이터 연동

백엔드

Python (FastAPI/Flask)

데이터 처리, AI 모델 구동 및 API 엔드포인트 구축

AI/분석

TensorFlow / Scikit-learn

시계열 데이터 기반 위험도 예측 모델 개발

프론트엔드

React / Vue.js

지도 기반 UI 및 사용자 인터페이스 구현

데이터베이스

PostgreSQL / Firebase Firestore

실시간 데이터 저장 및 관리

지도 API

Naver Maps API / Kakao Maps API

경남 해역 시각화 및 지역별 위험 등급 표시

🏃 로컬 설치 및 실행 (Installation)

이 프로젝트를 로컬 환경에서 실행하려면 다음 단계를 따르세요.

필수 요구사항

Node.js (v16+)

Python (v3.9+)

RISA API Key (API 연동 시)

1. 레포지토리 클론

git clone [YOUR_REPOSITORY_URL]
cd [YOUR_REPOSITORY_NAME]


2. 백엔드 설정 및 실행

# Python 가상 환경 설정
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 의존성 설치
pip install -r requirements.txt

# 서버 실행 (포트 8000 가정)
python main.py


3. 프론트엔드 설정 및 실행

cd frontend
npm install
npm run dev


(프론트엔드 설정 및 기술 스택에 따라 명령어를 수정해주세요.)

🧑‍🤝‍🧑 팀 (Team)

역할

이름

GitHub / Email

팀장/PM

강다현

[GitHub Profile Link]

팀원

[팀원 2 이름]

[GitHub Profile Link]

팀원

[팀원 3 이름]

[GitHub Profile Link]

팀원

[팀원 4 이름]

[GitHub Profile Link]

📄 라이선스 (License)

이 프로젝트는 [라이선스 타입 (예: MIT License)]에 따라 배포됩니다.
