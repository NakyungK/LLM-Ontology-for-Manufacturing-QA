# LLM-Ontology-for-Manufacturing-QA

# 🏭 KAMP 도금욕 공정 온톨로지 자동 구축 및 품질 분석 파이프라인

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/NakyungK/LLM-Ontology-for-Manufacturing-QA/blob/main/ontology_pipeline.ipynb)

> LLM 기반 온톨로지 자동 구축 방법론을 실제 제조 데이터에 적용하여  
> AI 품질 분석 리포트 품질 향상을 검증한 미니 프로젝트

---

## 📌 프로젝트 개요

네이버클라우드 AI 기획팀 인턴 과정에서 온톨로지 자동 구축 방법론을 리서치하며,  
**"LLM이 문서를 읽고 자동으로 온톨로지를 구축할 수 있는가?  
그리고 그 온톨로지가 실제 산업 현장에서 AI의 추론 품질을 높여줄 수 있는가?"**  
라는 질문을 실제 데이터로 검증하기 위해 설계한 프로젝트입니다.

---

## 🎯 핵심 질문

- LLM이 제조 공정 문서를 읽고 **사람 없이 온톨로지를 자동 구축**할 수 있는가?
- 자동 구축된 온톨로지를 컨텍스트로 제공하면 **AI의 품질 분석 리포트 품질이 향상**되는가?
- LLM 자동 구축의 **한계는 무엇이며, human-in-the-loop이 얼마나 중요한가?**

---

## 📊 데이터

| 항목 | 내용 |
|------|------|
| 데이터명 | KAMP 품질 이상탐지·진단(도금욕) AI 데이터셋 |
| 출처 | 중소벤처기업부, KAIST KAMP (kamp-ai.kr) |
| 제공기관 | KAIST (수행기관: ㈜에이비에이치/㈜임픽스) |
| 수집 기간 | 2021.09.06 ~ 2021.10.27 |
| 샘플 수 | 47,058건 (Lot 22개) |
| 변수 | pH, Temp, Current, Voltage, Result |
| 불량률 | 1.76% (불량 828건 / 정상 46,230건) |
| 라이선스 | 콘텐츠 변경허용 (출처 표기 필수) |

> 출처 표기: 중소벤처기업부, Korea AI Manufacturing Platform(KAMP),  
> 품질 이상탐지·진단(도금욕) AI 데이터셋, KAIST((주)에이비에이치,(주)임픽스),  
> 2021.12.27., www.kamp-ai.kr

---

## 🔬 방법론

### 참고 논문
- **Kommineni et al. (2024)** - "From human experts to machines: An LLM supported approach to ontology and knowledge graph construction" (arXiv:2403.08345)
- **Zengeya et al. (2026)** - "Axiom Generation for Automated Ontology Construction from Texts Through Schema Mapping" (Mach. Learn. Knowl. Extr.)

### 파이프라인 구조
```
[KAMP 가이드북 텍스트]
        ↓ GPT-4o 자동 생성
[Competency Questions (CQ) 10개]
        ↓ GPT-4o 자동 설계
[온톨로지 TBox: 클래스 + 관계 + 인과규칙]
        ↓ Human-in-the-loop
[정상범위 수정: 데이터 25%~75% 사분위수]
        ↓
[Lot별 온톨로지 규칙 위반 분석 (ABox)]
        ↓
┌─────────────────┬─────────────────────────┐
│   Naive RAG     │   온톨로지 기반 RAG         │
│ (데이터만 제공)     │ (데이터 + 온톨로지 제공)     │
└─────────────────┴─────────────────────────┘
        ↓ LLM Judge 평가
[품질 비교 수치 도출]
```

---

## 📁 파일 구조
```
├── ontology_pipeline.ipynb          # 전체 파이프라인 코랩 노트북
├── data/
│   └── kemp-abh-sensor.csv          # KAMP 도금욕 센서 데이터
├── ontology/
│   ├── 01_competency_questions.json # 자동 생성된 CQ 10개
│   ├── 02_tbox.json                 # 온톨로지 TBox
│   ├── 03_abox.json                 # Lot별 인스턴스 데이터 (ABox)
│   └── plating_ontology.ttl         # RDF/OWL 형식 온톨로지
└── results/
    ├── report_comparison.png        # 비교 실험 시각화
    └── quality_analysis_report.txt  # Lot별 리포트 전문
```

---

## 🧪 실험 결과

### LLM Judge 평가 결과 (정확성/구체성/인과관계/실용성, 각 10점 만점)

| Lot | Naive RAG | 온톨로지 기반 RAG | 우수 |
|-----|-----------|------------------|------|
| Lot 3 | 28/40 | 32/40 | 온톨로지 기반 |
| Lot 8 | 28/40 | 34/40 | 온톨로지 기반 |
| Lot 10 | 30/40 | 35/40 | 온톨로지 기반 |
| **평균** | **28.7/40** | **33.7/40** | **온톨로지 기반** |

### 핵심 수치

> **온톨로지 기반 RAG가 Naive RAG 대비 품질 분석 리포트 품질 15~17% 향상**  
> (LLM Judge의 확률적 특성으로 인해 실험마다 소폭 편차 존재)

### 평가 기준별 향상 내용

- **정확성**: 공정 도메인 지식 기반 수치 해석 가능
- **구체성**: 정상범위 이탈 건수/비율 등 구체적 수치 제시
- **인과관계**: `pH 이상 → 내구성 저하` 인과 체인 명시
- **실용성**: 변수별 구체적 권고사항 제시

---

## 💡 핵심 발견 및 인사이트

### 발견 1: 정상범위 설정이 온톨로지 품질을 결정한다

LLM이 자동 설정한 정상범위(데이터 전체 min~max)로 실험했을 때는 향상률이 **4.3%** 에 불과했으나, 데이터 사분위수(25% ~ 75%) 기반으로 수정 후 **15 ~ 17%** 로 향상됨.

→ 온톨로지는 구조만이 아니라 **그 안의 지식이 정확해야** 가치가 있음을 실험으로 확인.

### 발견 2: Human-in-the-loop의 필요성 확인

LLM이 CQ 생성 → TBox 설계 → 인과규칙 추출까지는 자동화 가능하나,  
**구체적인 수치 기준 설정은 반드시 전문가 검증이 필요**함.

→ Kommineni et al.(2024)이 제안한 **semi-automatic pipeline**의 필요성을 실험적으로 검증.

---

## ⚠️ 한계

1. **LLM Judge 객관성 문제**: 평가자도 LLM이라 온톨로지 기반 리포트를 편향적으로 평가했을 가능성 존재. 이상적으로는 도메인 전문가 평가 필요.

2. **정상범위 설정의 주관성**: 사분위수 기반 정상범위는 통계적 근거가 있으나 실제 공정 관리 기준과 다를 수 있음.

3. **단순 도메인의 한계**: 도금욕 공정은 변수가 4개로 단순하여 rule-based 시스템으로도 해결 가능. 온톨로지의 진가는 변수 간 관계가 복잡한 고도화된 공정에서 발휘될 것으로 판단.

4. **소규모 실험**: 3개 Lot을 대상으로 한 실험으로 통계적 유의성 확보에 한계 존재.

---

## 🔧 실행 환경

| 항목 | 내용 |
|------|------|
| 환경 | Google Colab |
| LLM | GPT-4o (OpenAI API) |
| 주요 라이브러리 | openai, rdflib, pandas, matplotlib |
| 온톨로지 뷰어 | Protégé (stanford.edu) |

---

## 🚀 실행 방법

1. Google Colab에서 `ontology_pipeline.ipynb` 열기
2. Colab Secrets에 `OPENAI_API_KEY` 등록
3. `kemp-abh-sensor.csv` 업로드
4. 셀 순서대로 실행

---

## 📚 References
```
Kommineni, V. K., König-Ries, B., & Samuel, S. (2024).
From human experts to machines: An LLM supported approach to ontology
and knowledge graph construction. arXiv:2403.08345.

Zengeya, T., Fonou-Dombeu, J. V., & Gwetu, M. (2026).
Axiom Generation for Automated Ontology Construction from Texts
Through Schema Mapping. Mach. Learn. Knowl. Extr., 8, 29.

중소벤처기업부, Korea AI Manufacturing Platform(KAMP),
품질 이상탐지·진단(도금욕) AI 데이터셋,
KAIST((주)에이비에이치,(주)임픽스), 2021.12.27., www.kamp-ai.kr
```
