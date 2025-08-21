# Malware Detection Extension

## 1. 개요

본 프로젝트는 웹 브라우저 확장 프로그램과 백엔드 분석 서버로 구성된 통합 보안 분석 시스템입니다. 사용자가 웹 서핑 중 마주치는 의심스러운 URL이나 파일을 클릭하면 실시간으로 분석하여 악성 여부를 판별하고, 상세 분석 리포트를 제공하는 것을 목표로 합니다.

## 2. 프로젝트 구조

본 시스템은 두 개의 독립적인 리포지토리로 구성되어 있습니다.

- **`chrome-extension/`**: 사용자 측(Client-side)에서 동작하는 Chrome 확장 프로그램입니다.
- **`server/`**: 분석 요청을 처리하는 백엔드 서버입니다.

## 3. 주요 기능

- **URL 분석**: 머신러닝 모델(XGBoost)을 기반으로 URL의 악성 가능성을 예측합니다.
- **파일 분석**: Yara 룰을 기반으로 파일의 악성 여부를 스캔합니다. 문서, 실행 파일, 압축 파일 등 다양한 파일 유형을 지원합니다.
- **리포트 생성**: 분석 결과를 바탕으로 JSON 형식의 로그와 PDF 형식의 상세 리포트를 생성합니다.

## 4. 시스템 흐름

1.  **요청 (Chrome Extension)**: 사용자가 웹 페이지에서 분석하고 싶은 URL이나 파일을 선택하고 확장 프로그램을 통해 분석을 요청합니다.
2.  **전송 (Chrome Extension → Server)**: 확장 프로그램은 분석 대상을 백엔드 서버의 API로 전송합니다.
3.  **분석 (Server)**:
    - 서버는 요청받은 데이터가 URL인지 파일인지 식별합니다.
    - URL인 경우, `url_analyzer`가 머신러닝 모델을 사용해 위험도를 분석합니다.
    - 파일인 경우, `yara_scan`이 `rules` 디렉토리에 정의된 Yara 룰셋을 기반으로 파일을 스캔합니다.
4.  **결과 처리 및 리포트 생성 (Server)**:
    - `analysis_engine`이 분석 결과를 종합합니다.
    - 분석 결과는 `reports/` 디렉토리에 JSON 파일로 저장됩니다.
    - 필요한 경우, `pdf_report` 모듈이 상세 PDF 리포트를 생성합니다.
5.  **응답 (Server → Chrome Extension)**: 서버는 분석 결과를 요약하여 확장 프로그램에 다시 전송합니다.
6.  **결과 확인 (Chrome Extension)**: 사용자는 확장 프로그램의 UI를 통해 분석 결과를 즉시 확인할 수 있습니다.

## 5. 리포지토리 상세

### 5.1. `chrome-extension`

브라우저에서 직접 실행되는 클라이언트입니다.

- **`manifest.json`**: 확장 프로그램의 설정 파일입니다.
- **`background.js`**: 백그라운드에서 실행되며, 서버 통신 등 핵심 로직을 처리합니다.
- **`content-script.js`**: 현재 웹 페이지의 DOM에 접근하여 컨텍스트 메뉴 등을 제어합니다.

### 5.2. `server`

Python과 Flask를 기반으로 구축된 분석 서버입니다. Docker를 통해 컨테이너 환경에서 실행됩니다.

- **`flask_server.py` / `controller.py`**: API 엔드포인트를 정의하고 요청을 분배하는 웹 서버 파트입니다.
- **`analysis_engine.py`**: URL, 파일 분석 등 핵심 분석 로직을 관장하는 메인 엔진입니다.
- **`url_analyzer.py`**: `xgb_url_classifier.joblib` 모델을 로드하여 URL을 분석합니다.
- **`yara_scan.py`**: Yara 룰을 이용해 파일을 스캔합니다.
- **`rules/`**: 기능별/파일 유형별로 분류된 Yara 룰셋이 저장된 디렉토리입니다.
- **`reports/`**: 모든 분석 결과(JSON, PDF)가 저장되는 디렉토리입니다.
- **`Dockerfile` / `docker-compose.yml`**: 서버를 Docker 컨테이너로 빌드하고 실행하기 위한 설정 파일입니다.

## 6. 팀원

| 이름     | 역할               | GitHub                                   |
|----------|--------------------|------------------------------------------|
| 서범창 | PM / YARA rule 관리 | [@west-window](https://github.com/west-window) |
| 문석환 | 악성코드 분석 서버 구축 | [@noc02](https://github.com/noc02)   |
| 박상경 | URL ML 구축 | [@bsk2002](https://github.com/bsk2002)   |
| 김혜지 | 데이터셋 확보, 보고서 구성 | [@hyeroro](https://github.com/hyeroro)   |

---
