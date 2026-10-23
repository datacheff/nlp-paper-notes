# IterCQR: Iterative Conversational Query Reformulation with Retrieval Guidance

> AI가 원문을 대조해 준비한 학습 자료. 사용자의 정독·실험 기록이 아니다.

| 항목 | 내용 |
|---|---|
| 저자 | Yunah Jang, Kang-il Lee, Hyunkyung Bae, Hwanhee Lee, Kyomin Jung |
| 학회 / 연도 | NAACL 2024 Long Papers, pp.8121–8138 |
| 출판본·DOI | [ACL 공식 페이지](https://aclanthology.org/2024.naacl-long.449/) · 10.18653/v1/2024.naacl-long.449 |
| 원문 | [공식 PDF](https://aclanthology.org/2024.naacl-long.449.pdf) · [로컬 PDF](./original.pdf) · [저자 HTML v2](https://arxiv.org/html/2311.09820v2); 기준은 출판 PDF |
| 공식 코드 | [YunahJang/IterCQR](https://github.com/YunahJang/IterCQR), PDF 첫 페이지 각주·README에서 확인 |
| 읽은 날짜 / 자료 수정일 | 사용자 미시작 / 2026-10-23 |
| 읽기 / 구현·재현 | **미시작 / 미시도** |
| 자료 상태 / 검토 기준일 | **자료 준비됨 / 2026-10-23** |

## 1. 핵심 요약 / Takeaway

- **문제:** 사람이 이해하기 좋은 재작성 질의가 검색기에도 최적이라는 보장은 없다(§1).
- **방법:** LLM 재작성으로 초기화한 생성기를 검색 보상으로 반복 학습한다. 초기 MBR 탐색 후 최고 후보 NLL 학습으로 전환한다(§3.2, Fig.2).
- **결과:** TopiOCQA BM25 MRR은 ConvGQR 0.124→IterCQR 0.165, QReCC는 0.441→0.467(Table1).
- **AI 해석:** 문장 정답 모방 대신 검색에 유용한 문장을 배우는 사례다. 응답 생성 전체를 평가한 논문은 아니다.

## 2. 입력·출력과 선수 지식 / Task & prerequisites

| 항목 | 내용·설명용 예 | 근거 |
|---|---|---|
| 입력 | 이전 질문·답변과 현재 질문: “후버댐 …”→“어디 있어?” | §3.1 |
| 출력 | 독립 질의: “후버댐은 어디에 있는가?” | Fig.1 |
| 목표 | 고정 검색기로 관련 passage를 상위 검색 | §3.1–3.3 |

실제 출력이 아닌 설명용 예다. 선수 개념은 T5, BM25, ANCE, 기대 보상, MRR이다. CQR은 RAG 앞단에 결합할 수 있다.

## 3. 핵심 방법 / Method

흐름: 대화→LLM 초기 rewrite→T5-base→후보 10개→정답 passage와 ANCE cosine 보상→생성기 갱신.

τ=1이면 M0는 초기 NLL, M1은 MBR, M2 이후는 보상 최고 후보 NLL이다. 후보는 반복마다 재생성하고 해당 반복 학습 중에는 고정한다(Algorithm1). Eq.2는 후보 재정규화 확률로 가중한 음의 기대 보상이며, Eq.3의 cosine은 min-max 정규화한다. 상세 수식은 [상세 해설](./detailed-explanation.md)에 있다.

학습에서 **gold passage annotation은 필요**하다. 사람 rewrite가 없다는 뜻을 완전 무감독으로 확대하지 않는다. ANCE는 고정하며 추론은 최종 생성 질의→검색이다(§2.1, §3).

## 4. 실험과 근거 / Evidence

Table1의 MRR(↑), 분산 미보고·Appendix A 단일 run:

| 데이터 | 검색기 | ConvGQR | IterCQR |
|---|---|---:|---:|
| TopiOCQA | ANCE | 0.256 | 0.263 |
| TopiOCQA | BM25 | 0.124 | 0.165 |
| QReCC | ANCE | 0.420 | 0.429 |
| QReCC | BM25 | 0.441 | 0.467 |

영어 QReCC·TopiOCQA를 사용한다(§4, Ethical Statement). CONQRR dense는 ANCE가 아닌 DualEncoder여서 직접 동일 조건 비교가 아니다(Table1 주석). QReCC BM25 R@100은 IterCQR 85.5, ConvGQR 88.0으로 모든 지표 우위는 아니다.

절제실험 TopiOCQA ANCE MRR: IterCQR 0.263, OnlyMBR 0.216, OnlyTop1 0.248(Table4). 이 설정의 단계 결합을 지지한다. split별 건수·별도 누수 검사는 미확인이다.

## 5. 한계·비판·질문 / Limits & questions

- **저자 한계:** 초기 LLM 비용·성능 의존, 후보 저장 공간·긴 학습 시간, dense 보상의 sparse 검색 개선 불확실성(Limitations).
- **AI 검토 질문:** cosine 상승이 실제 순위 상승과 어긋나는 질의는 무엇인가? §5의 요약 학습 해석은 overlap 분석에 기반한다.
- **사용자 비판·이해 불확실성:** 미작성. 이해·동의 여부는 정독 후 기록한다.

## 6. 인턴십 연구로 확장 / Research seed

**AI 제안·미검증 / 신규성 미확인:** 후보 수 10을 3·5로 줄이면 고정 GPU 시간에서 성능·비용 균형이 좋아지는가? 같은 데이터와 seed에서 MRR·R@100·총 생성/학습 시간을 비교한다. 단순 후보 수 ablation을 넘어서는 기여인지 선행연구 검토가 필요하다.

3개월 후보 계획: 1개월 baseline·자원 측정→2개월 한 변수 변경→3개월 반복·오류 분석. 사용자 GPU·예산은 미확인이며 실행 약속이나 성과가 아니다. 다음 행동은 환경 요구사항과 공개 checkpoint 접근 확인이다.

## 7. 연결할 읽을거리와 검증 / Follow-up

[쉬운 설명](./easy-explanation.md) · [상세 해설](./detailed-explanation.md) · [퀴즈](./quiz.md) · [선행논문 해설](./references.md)

- [CONQRR, EMNLP 2022](https://aclanthology.org/2022.emnlp-main.679/): 검색 보상 학습 비교(§2.1). 원문은 rewrite 무감독 변형도 포함한다.
- [ConvGQR, ACL 2023](https://aclanthology.org/2023.acl-long.274/): 재작성+예상 답변 확장과 비교.
- [Smith & Eisner, 2006](https://aclanthology.org/P06-2101/): 기대 손실 최적화의 기반.
- [MILAB 목록](https://milab.snu.ac.kr/publications): 2024 등재 확인.

AI는 서지·수식·표를 원문 대조했다. 아래는 **사용자 확인란**이며 완료를 대신 표시하지 않았다.

- [ ] 원문에서 핵심 주장·비교 조건을 직접 확인했다.
- [ ] 수식·한계를 내 말로 설명하고 개인 의견을 적었다.
- [ ] 읽기 완료와 실제 재현 상태를 각각 갱신했다.

재현 로그: 미실행; 환경·commit·관측값 없음.

출처: Jang et al. (2024), ACL, 위 출판본. 로컬 PDF는 원문 무변경 보관본이다. 원문은 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)이며 이 노트는 출처를 밝힌 한국어 해설·재구성이다.
