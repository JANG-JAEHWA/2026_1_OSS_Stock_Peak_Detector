# 1. Conceptualization

**Project Title:** 주식 고점 판독기 (Stock Peak Detector)  
<img width="2816" height="1536" alt="Gemini_Generated_Image_p4nx72p4nx72p4nx" src="https://github.com/user-attachments/assets/e39d17ed-4bba-4340-ba9b-3dc6c957045d" />

**22110615 / 장재화 / jaehwajang@yu.ac.kr**

---

### [ Revision history ]

| Revision date | Version # | Description | Author |
| :--- | :--- | :--- | :--- |
| 2026.03.27 | 1.0.0 | Conceptualization Document 상세 초안 작성 | 장재화 |

---

### = Contents =
1. [Business purpose](#1-business-purpose)
2. [System context diagram](#2-system-context-diagram)
3. [Use case list](#3-use-case-list)
4. [Concept of operation](#4-concept-of-operation)
5. [Problem statement](#5-problem-statement)
6. [Glossary](#6-glossary)
7. [References](#7-references)

---

## 1. Business purpose

### 1.1. Project background
주식 시장에서 매도 타이밍을 잡는 것은 매수하는 것만큼 매우 어려우며, 특히 주가가 급등하여 최고점에 다다랐을 때 언제 매도해야 할지 모를 경우가 많습니다.
기존의 단순한 기술적 보조지표(RSI, MACD 등)만으로는 시장의 복잡한 흐름과 패턴을 완벽히 읽어내기 어렵습니다. 
따라서 과거의 방대한 차트 흐름(시계열 데이터)을 스스로 학습하여, 현재 주가의 위치가 과거의 '고점 패턴'과 얼마나 일치하는지 객관적인 확률로 알려주는 딥러닝 기반의 시스템이 필요합니다.

### 1.2. Goal
* 한국 주식 시장(코스피/코스닥)의 시가총액 상위 종목의 최소 10년 치 과거 OHLCV(시가, 고가, 저가, 종가, 거래량) 데이터를 수집 및 구축합니다.
* 딥러닝 시계열 분석 모델인 LSTM(Long Short-Term Memory)을 활용하여 주가 고점 패턴을 학습시킵니다.
* 사용자가 특정 종목을 조회하면, 시스템이 해당 종목의 최근 차트 흐름을 분석하여 '현재 고점일 확률(%)'을 수치화하여 제공합니다.

### 1.3. Target Market
* 감정적인 뇌동매매를 배제하고 데이터와 패턴에 기반한 이성적인 투자를 지향하는 한국 주식 시장 개인 투자자.
* 투자 결정 시 객관적으로 참고할 수 있는 하나의 보조 지표로 활용하고자 하는 투자자

---

## 2. System context diagram

*다음 그림은 시스템의 전체적인 데이터 흐름과 구성 요소 간의 관계를 나타냅니다.
<img width="2816" height="1536" alt="Gemini_Generated_Image_7hcq0j7hcq0j7hcq" src="https://github.com/user-attachments/assets/5503885c-999f-41a9-a9d4-4401614f6325" />


* **User (사용자):** 프로그램(터미널)에 접속하여 종목을 검색하고 고점 확률 데이터를 확인합니다.
* **System (주식 고점 판독 시스템):**
    * **Console Interface:** 사용자의 검색 요청을 받고 분석 결과를 터미널(콘솔) 화면에서 텍스트로 보여줍니다.
    * **AI Prediction module:** 수집된 데이터를 스케일링(전처리)하고 LSTM 모델에 입력하여 고점 확률을 추론(Inference)합니다.
* **Local File Storage (로컬 파일 저장소):** 별도의 DB 엔진 없이, 종목별 과거 주가 데이터(CSV 형식)와 학습된 AI 모델의 가중치(H5 또는 PTH 파일)를 디렉토리에 저장하고 관리합니다.
* **External Data API (외부 데이터 제공자):** FinanceDataReader 등을 통해 최신 주가 데이터를 요청하고, 이를 시스템이 로컬 파일(CSV) 형태로 저장 및 업데이트합니다.

---

## 3. Use case list

| Use Case | Actor | Description |
| :--- | :--- | :--- |
| **1) 종목 검색 및 분석 요청** | User | 사용자가 코스피/코스닥에 상장된 특정 종목명 또는 종목 코드를 터미널 입력창에 타이핑하여 고점 분석을 요청합니다. |
| **2) 고점 확률 결과 조회** | User | 시스템이 반환한 해당 종목의 현재 고점 위험도(%) 결과를 터미널 화면에서 확인합니다. |
| **3) 주가 데이터 수집 및 전처리** | System | 외부 API로부터 최신 주가 데이터를 수집하고, 딥러닝 모델이 인식할 수 있도록 데이터를 정규화하여 로컬 파일(CSV)로 저장합니다. |
| **4) 딥러닝 모델 추론(Inference)** | System | 로컬 폴더에 저장된 특정 종목의 최근 N일치 시계열 데이터를 불러와, 사전 학습된 LSTM 모델 파일에 통과시켜 고점 확률을 계산합니다. |

---

## 4. Concept of operation

**1) 종목 검색 및 분석 요청**
| 항목 | 내용 |
| :--- | :--- |
| **Purpose** | 사용자가 분석을 원하는 특정 주식 종목을 시스템에 전달하기 위함입니다. |
| **Approach** | 터미널 프롬프트에 종목명이나 코드를 입력하면, 내부 종목 리스트 파일(CSV)을 조회하여 정확한 종목 코드를 매칭합니다. |
| **Dynamics** | 사용자가 투자 전 특정 종목의 현재 위치(고점 여부)가 궁금할 때 실행합니다. |
| **Goals** | 사용자의 입력을 정확한 종목 코드(예: 삼성전자 -> 005930)로 변환하여 내부 분석 모듈로 전달합니다. |

**2) 딥러닝 모델 추론 (Inference)**
| 항목 | 내용 |
| :--- | :--- |
| **Purpose** | 입력된 종목의 최근 차트 흐름을 분석하여 고점 확률을 도출하기 위함입니다. |
| **Approach** | 프로그램은 해당 종목의 최근 60일(또는 120일) 치 OHLCV 데이터를 CSV 파일에서 불러옵니다. 이 데이터를 0과 1 사이의 값으로 스케일링(MinMaxScaler)한 후, 디스크에 저장된 LSTM 모델 가중치를 로드하여 0~100% 사이의 결과값을 도출합니다. |
| **Dynamics** | 사용자의 검색 입력이 완료된 직후 연산이 실행됩니다. |
| **Goals** | 파일 읽기, 데이터 전처리 및 AI 연산을 신속하게 수행하여 터미널 화면에 결과를 출력합니다. |

---

## 5. Problem statement

### 5.1. Overview
 이 시스템은 시계열 데이터를 다루는 딥러닝 모델을 기반으로 하므로, 데이터의 품질과 모델의 일반화 능력이 프로젝트의 성패를 좌우합니다. 
초기 개발 및 운영의 복잡성을 줄이기 위해 무거운 RDBMS(관계형 데이터베이스) 대신 로컬 파일(CSV 및 모델 파일) 기반 구조로 데이터를 저장하고 관리합니다.

### 5.2. Problem definition

* **Problem #1: 종목 간 주가 편차 및 데이터 스케일 문제 (Data Scaling)**
  * **설명:** 종목마다 주가의 단위(만원 단위, 십만원 단위 등)가 다릅니다. 딥러닝 모델에 이 가격을 그대로 넣으면 주가가 높은 종목에 가중치가 잘못 쏠리는 현상이 발생합니다.
  * **해결책:** 모든 종목의 주가 데이터를 0과 1 사이의 값으로 변환하는 정규화(MinMaxScaler) 전처리 과정을 반드시 거친 후 모델에 학습 및 입력해야 합니다.

* **Problem #2: 과적합 (Overfitting)**
  * **설명:** AI 모델이 과거의 특정 상승장 패턴에만 너무 완벽하게 맞춰져서, 전혀 다른 흐름의 하락장이나 횡보장에서는 예측력이 현저히 떨어질 수 있습니다.
  * **해결책:** 학습 데이터에 드롭아웃(Dropout) 층을 추가하여 모델의 과의존성을 줄이고, 10년 치 데이터 중 상승장, 하락장, 폭락장(예: 코로나 시기)을 모두 포함하여 균형 있게 학습시킵니다.

### 5.3. NFRs (Non-Functional Requirements)
* ① 사용자 경험을 위해 종목 검색 후 AI 추론 결과가 반환되기까지의 지연 시간은 최대 3초 이내여야 합니다.
* ② 프로그램은 파이썬(Python) 로컬 환경에서 구축하며, 딥러닝 프레임워크는 TensorFlow(Keras) 또는 PyTorch를 사용합니다. 데이터는 CSV 형태로 디렉토리 내에 체계적으로 관리합니다.

---

## 6. Glossary

| Term | Description |
| :--- | :--- |
| **고점 (Peak)** | 본 프로젝트에서는 초기 학습을 위해 임시로 **'특정일 기준으로 향후 5일 이내에 주가가 10% 이상 하락하는 시점'**으로 정의하며, 추후 모델 학습 결과를 바탕으로 이 기준을 정교하게 수정 및 보완할 예정입니다. |
| **LSTM (Long Short-Term Memory)** | 과거의 데이터를 순차적으로 처리하며, 중요한 과거 정보는 오래 기억하고 불필요한 정보는 잊어버리도록 설계된 인공신경망. 주가 차트 흐름 같은 시계열 데이터 분석에 탁월합니다. |
| **OHLCV** | 주식 차트를 구성하는 5가지 기본 데이터 (Open: 시가, High: 고가, Low: 저가, Close: 종가, Volume: 거래량) |
| **MinMaxScaler (정규화)** | 데이터의 최솟값을 0, 최댓값을 1로 변환하여 데이터의 단위를 통일하는 전처리 기법입니다. 딥러닝 학습 시 필수적입니다. |
| **Inference (추론)** | 이미 학습이 완료된 AI 모델에 새로운 데이터를 입력하여 결과를 예측하고 도출해 내는 과정을 의미합니다. |

---

## 7. References
1. [FinanceDataReader](https://github.com/FinanceData/FinanceDataReader) (한국 주식 데이터 수집 라이브러리)
2. TensorFlow / PyTorch Official Documentation (딥러닝 모델 구현 참고)
