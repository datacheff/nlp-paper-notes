# MILAB 논문 읽기 목록

기준일은 **2026-10-23**이다. [서울대학교 MILAB publications](https://milab.snu.ac.kr/publications)에 있는 **2022~2026년의 대화·요약·질의응답 관련 논문 중 12편을 선별**했다. 전체 논문의 전수 분류나 중요도 순위가 아니다. 여기서 최근 5개년은 2022·2023·2024·2025·2026의 다섯 달력 연도이며, 기준일 이후에만 공개되는 자료는 포함하지 않는다.

첫 상세 학습자료 대상은 [IterCQR](papers/2024-itercqr/README.md)이다. 자료의 작성 여부와 사용자의 학습 진도는 구분한다. **아래 모든 논문의 사용자 읽기 상태는 미시작, 구현·재현은 미시도**다. 다른 논문들의 완전한 정독 노트까지 준비되었다는 의미는 아니다.

## 연도와 출처를 기록하는 기준

- 목록의 연도와 폴더명은 학회·워크숍 **출판연도**를 사용한다. arXiv 최초 공개연도는 별도로 기록한다. 예를 들어 IterCQR는 arXiv 2023년, NAACL 출판 2024년이다.
- 제목·venue·쪽수는 연구실 소개 페이지보다 **ACL Anthology의 정식 원문**을 우선한다. 저자 저장소의 인용문에도 초기 서지정보가 남아 있을 수 있다.
- 2026년 PRAGMA와 CompCQR는 공개 arXiv 원문과 저자의 accepted 표기를 확인했다. 이 표에서는 proceedings 출판 여부를 별도로 단정하지 않는다.
- `별도 미확인`은 이번 조사에서 arXiv 레코드를 확인하지 않았다는 뜻이다. arXiv가 존재하지 않는다는 뜻이 아니다.
- 선행논문은 MILAB 소속·최근 5개년 제한의 대상이 아니다. 따라서 IterCQR 이해에 필요한 2006년 논문도 읽는다.

## 선별 12편

권장 순서는 지금 선택한 첫 논문에서 연결되는 흐름을 기준으로 했다. 각 제목 링크는 공식 원문 페이지다. arXiv 열의 연도도 해당 레코드에 연결된다.

| 순서 | 출판연도·venue / 공개 상태 | 정확한 제목 · 공식 원문 | arXiv 최초 공개 | 태스크와 읽을 지점 | 사용자 읽기 |
| --- | --- | --- | --- | --- | --- |
| 1 | 2024 · NAACL | [IterCQR: Iterative Conversational Query Reformulation with Retrieval Guidance](https://aclanthology.org/2024.naacl-long.449/) | [2023](https://arxiv.org/abs/2311.09820) | 대화형 검색. 검색 보상으로 재작성 모델을 반복 학습. 첫 [상세 학습자료](papers/2024-itercqr/README.md) 대상 | 미시작 |
| 2 | 2026 · Findings of EMNLP accepted, arXiv 공개 확인 | [CompCQR: Compositional Query Generation for Training-Free Conversational Search](https://arxiv.org/abs/2609.14646) | 2026 | 대화형 검색. 구성요소를 조합하는 training-free 질의 생성과 검색 비용 비교 | 미시작 |
| 3 | 2025 · COLING | [Return of EM: Entity-driven Answer Set Expansion for QA Evaluation](https://aclanthology.org/2025.coling-main.743/) | [2024](https://arxiv.org/abs/2404.15650) | QA 평가. 정답의 다양한 표면형과 soft EM; 무엇을 성공으로 측정할지 점검 | 미시작 |
| 4 | 2023 · Findings of EMNLP | [Asking Clarification Questions to Handle Ambiguity in Open-Domain QA](https://aclanthology.org/2023.findings-emnlp.772/) | [2023](https://arxiv.org/abs/2305.13808) | 모호성 탐지 → 명확화 질문 → 후속 QA; CAmbigNQ의 문제 정의 | 미시작 |
| 5 | 2022 · DialDoc workshop | [Improving Multiple Documents Grounded Goal-Oriented Dialog Systems via Diverse Knowledge Enhanced Pretrained Language Model](https://aclanthology.org/2022.dialdoc-1.15/) | 별도 미확인 | 여러 문서에 근거한 대화. 근거 span 예측과 응답 생성의 연결 | 미시작 |
| 6 | 2023 · EMNLP | [Dialogizer: Context-aware Conversational-QA Dataset Generation from Textual Sources](https://aclanthology.org/2023.emnlp-main.545/) | [2023](https://arxiv.org/abs/2311.07589) | 문서를 ConvQA 합성 데이터로 바꾸기. 질문과 답의 정합성·문맥 관련성 | 미시작 |
| 7 | 2024 · EMNLP | [MP2D: An Automated Topic Shift Dialogue Generation Framework Leveraging Knowledge Graphs](https://aclanthology.org/2024.emnlp-main.979/) | [2024](https://arxiv.org/abs/2403.05814) | 지식 그래프와 여러 passage로 주제 전환 대화 생성; TS-WikiDialog | 미시작 |
| 8 | 2022 · Findings of NAACL | [Masked Summarization to Generate Factually Inconsistent Summaries for Improved Factual Consistency Checking](https://aclanthology.org/2022.findings-naacl.76/) | [2022](https://arxiv.org/abs/2205.02035) | 요약 사실성 탐지. 마스킹으로 그럴듯한 오류 요약을 만드는 MFMA | 미시작 |
| 9 | 2022 · GEM workshop | [Factual Error Correction for Abstractive Summaries Using Entity Retrieval](https://aclanthology.org/2022.gem-1.41/) | [2022](https://arxiv.org/abs/2204.08263) | 요약 사실성 교정. 근거 문장 검색 후 entity 오류를 탐지·교체하는 RFEC | 미시작 |
| 10 | 2023 · ACL | [BREAK: Breaking the Dialogue State Tracking Barrier with Beam Search and Re-ranking](https://aclanthology.org/2023.acl-long.159/) | 별도 미확인 | 목적 지향 대화의 상태 추적. 후보 생성과 reranking, MultiWOZ 평가 | 미시작 |
| 11 | 2025 · NAACL | [AdvisorQA: Towards Helpful and Harmless Advice-seeking Question Answering with Collective Intelligence](https://aclanthology.org/2025.naacl-long.333/) | [2024](https://arxiv.org/abs/2404.11826) | 주관적 조언 QA. 하나의 정답 대신 helpfulness ranking과 harmlessness 평가 | 미시작 |
| 12 | 2026 · EMNLP accepted, arXiv 공개 확인 | [PRAGMA: Evaluating Personalized Guidance with Memory Alignment in Lifelong Conversations](https://arxiv.org/abs/2609.09664) | 2026 | 장기 대화 기억에 근거한 개인화 조언. 검색 성공과 응답의 grounding을 구분 | 미시작 |

PRAGMA는 2026-09-09에 처음 공개되었고, 09-23의 v2에서 Table 5의 일부 RAG 결과를 수정했다. 이 목록에서는 [v2](https://arxiv.org/abs/2609.09664v2)를 읽는다. CompCQR의 최초 공개일은 2026-09-13이다. 두 레코드 모두 위 기준일 이전에 공개되었으며 해당 학회 accepted 표기를 포함한다. [PRAGMA 공개 이력](https://arxiv.org/abs/2609.09664), [CompCQR 공개 이력](https://arxiv.org/abs/2609.14646)

## 첫 논문 이후의 읽기 경로

처음부터 12편을 연속 정독하기보다, IterCQR를 설명할 수 있게 된 뒤 한 가지 경로를 고른다. 아래 연결은 학습을 위한 제안이며 논문 간 직접 후속관계를 모두 뜻하지 않는다.

```text
IterCQR (첫 학습자료)
  ├─ CONQRR (2022): 검색 보상을 사용한 재작성의 선행 접근
  ├─ ConvGQR (2023): rewrite + 예상 답변 확장이라는 비교 접근
  ├─ Smith & Eisner (2006): 기대손실 최소화의 수학적 배경
  └─ CompCQR (2026): training-free 대화 검색으로 비용·성능 비교

대화 검색에서 문제를 확장한다면
  ├─ Asking Clarification Questions → Return of EM (모호성·평가)
  ├─ 문서 근거 대화 → Dialogizer → MP2D (데이터·주제 전환)
  ├─ AdvisorQA → PRAGMA (주관적 조언·장기 기억)
  └─ BREAK (목적 지향 대화의 상태 추적)

요약 사실성에 관심이 더 크다면
  └─ Masked Summarization / MFMA → RFEC (오류 탐지에서 교정으로)
```

첫 세 선행논문은 [IterCQR 핵심 참고논문 카드](papers/2024-itercqr/references.md)에 문제·원리·차이·확인 질문을 정리했다. **카드를 읽는 것과 그 논문의 전체 정독·재현을 완료하는 것은 별개**다. CONQRR 자체에도 zero rewrite supervision 실험이 있으므로, IterCQR를 “최초로 사람 rewrite를 없앤 연구”라고 요약하지 않는다.

3개월 인턴 연구로 좁힐 때에는 `한 데이터셋 + 한 기준 모델 + 바꿀 요소 하나 + 평가 지표`를 먼저 정한다. 예를 들어 IterCQR의 후보 다양성·보상·학습 비용 중 하나만 변경하고, CompCQR와는 같은 retriever·분할에서 비교하는 방식이다. 구체적인 가설은 [연구 아이디어](research-ideas.md)에 따로 기록한다.

## 추가 후보 — 아직 순서를 정하지 않음

선별 12편 밖에도 직접 관련되거나 인접한 논문이 있다. 아래 역시 사용자 읽기 상태는 **미시작**이다.

| 출판연도·venue | 정확한 제목 · 공식 원문 | 연결할 수 있는 질문 | arXiv 구분 |
| --- | --- | --- | --- |
| 2023 · Findings of ACL | [Injecting Comparison Skills in Task-Oriented Dialogue Systems for Database Search Results Disambiguation](https://aclanthology.org/2023.findings-acl.249/) | 여러 DB 검색 결과를 비교해 사용자 선택을 돕는 대화. 명확화 질문·BREAK 경로와 연결 | 별도 미확인 |
| 2023 · DSTC11 workshop | [Leveraging Ensemble Techniques and Metadata for Subjective Knowledge-grounded Conversational Systems](https://aclanthology.org/2023.dstc-1.23/) | 주관적 지식 선택과 응답 생성. AdvisorQA·문서 근거 대화 경로와 연결 | 별도 미확인 |
| 2026 · ACL로 MILAB에 등재; arXiv 원문 확인 | [How Training Data Shapes the Use of Parametric and In-Context Knowledge in Language Models](https://arxiv.org/abs/2510.02370) | 모델 내부 지식과 문맥 지식의 사용. QA·RAG의 배경 분석으로 확장 | 2025 최초 공개; 학회 등재연도와 구분 |

마지막 후보는 이번 조사에서 MILAB의 ACL 2026 등재와 공개 arXiv 원문을 확인했다. 정식 ACL Anthology 레코드는 별도 확인 대상으로 남긴다. 원문 접근이 확인되지 않은 제목을 이미 읽을 수 있는 논문처럼 추가하지 않는다. 멀티모달·음성·스토리 생성 등은 이번 첫 선별의 중심 범위 밖에 두었다.

## 공식 구현으로 넘어갈 때

아래는 논문이 제시한 저자 구현·데이터 링크다. 링크 확보만 한 상태이며 설치·실행·접근 권한·현재 재현 가능성은 검증하지 않았다.

- [IterCQR](https://github.com/YunahJang/IterCQR), [CompCQR](https://github.com/YunahJang/CompCQR)
- [Return of EM / ENTQA](https://github.com/DongryeolLee96/ENTQA)
- [MFMA](https://github.com/hwanheelee1993/MFMA), [RFEC](https://github.com/hwanheelee1993/RFEC)
- [BREAK](https://github.com/tony-won/DST-BREAK)
- [PRAGMA 코드](https://github.com/yuhyojeong/PRAGMA), [PRAGMA 데이터](https://huggingface.co/datasets/stellahj/PRAGMA)

다음 논문을 골랐을 때 해당 원문의 정확한 버전과 평가 조건을 다시 확인하고, 논문별 폴더와 실제 학습 기록을 한 편씩 추가한다.
