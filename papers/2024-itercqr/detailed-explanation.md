# IterCQR 상세 해설 — AI 다른 분야 석사 독자를 위한 읽기

> **자료 준비됨 / 사용자 학습 미시작 / 구현·재현 미시도.** 검토 기준일 2026-10-23. 원문의 사실, AI의 수학·교육용 해설, AI 연구 가설을 구분했다. 아래의 예제 수치는 별도 표시가 없으면 설명용이며 실험 관측값이 아니다. 사용자의 이해·비판·배운 점은 아직 작성하지 않았다.

원저자: Yunah Jang, Kang-il Lee, Hyunkyung Bae, Hwanhee Lee, Kyomin Jung. NAACL 2024, pp.8121–8138. [ACL 출판본](https://aclanthology.org/2024.naacl-long.449/) · [공식 PDF](https://aclanthology.org/2024.naacl-long.449.pdf) · [로컬 원문 PDF](./original.pdf) · [저자 HTML v2](https://arxiv.org/html/2311.09820v2) · [공식 코드](https://github.com/YunahJang/IterCQR). HTML은 2024-04-08 preprint이며 수치·페이지 기준은 출판 PDF다. 코드 공식성은 PDF 각주·README로 확인했다.

## 1. 핵심 요약 / Takeaway

“후버댐의 전력은 어디로 가는가?” 다음 “그건 어디에 있어?”를 검색하면 대상을 알기 어렵다. CQR(conversational query reformulation)은 이를 문맥 없는 독립 질의로 바꾼다. 이 한국어 예는 Fig.1을 설명하는 재구성이며 실제 모델 출력이 아니다.

사람의 독립 질문을 정답으로 모방해도 검색에는 최적이 아닐 수 있다. 저자는 LLM 재작성으로 시작해 검색에 유용한 후보의 생성 확률을 높인다(§1, §3.2).

**AI의 읽기 관점:** “정답 문장을 그대로 모방하는 손실”과 “최종 사용처에 도움이 되는 문장을 만드는 손실”의 차이를 배우기 좋다. 다만 이 논문만으로 대화 생성·환각·최종 답변 정확도를 공부했다고 보기는 어렵다. 주된 증거는 passage 검색 순위 지표이다.

## 2. 입력·출력과 선수 지식 / Task & prerequisites

### CQR과 RAG를 같은 층에 놓지 않기

§3.1에서 입력은 현재 질문 $q_k$와 이전 질문·답변 $H_{k-1}$이고, 이전 turn은 역순으로 이어 붙인다. 생성기 $M$의 출력 $q_k^*$는 문자열이며 검색기 $R$의 입력이 된다. 학습용 정답 passage $p_i$는 별도 감독 정보다.

AI가 그린 시스템 위치는 다음과 같다.

대화 이력·현재 질문 → **CQR 질의 생성** → 문서 검색 → 검색 문서를 읽는 답변 생성기 → 최종 답변

IterCQR은 굵은 모듈을 학습하고 검색을 평가한다. RAG는 검색 문서를 조건으로 답변을 생성한다([Lewis et al., 2020](https://arxiv.org/abs/2005.11401)). CQR을 RAG 앞단에 결합할 수 있지만, MRR 상승이 답변의 사실성 향상과 같지는 않다. IterCQR은 이 전체 파이프라인을 평가하지 않았다.

### BM25와 ANCE가 “질문을 읽는” 방식

| 개념 | AI의 설명 | IterCQR에서의 역할 |
|---|---|---|
| BM25 | 질문과 문서의 단어 일치를 IDF·빈도 포화·문서 길이로 가중한다. 고유명사·핵심어 복원이 도움이 될 수 있다. | 희소 검색 평가 |
| ANCE | 질의와 passage를 학습된 밀집 벡터 공간에 놓고 검색한다. 의미가 가까워도 정답 순위가 보장되지는 않는다. | 밀집 검색 평가 및 보상 계산용 고정 인코더 |
| T5-base | 입력 텍스트를 읽는 encoder와, 앞서 생성한 토큰을 조건으로 다음 토큰을 내는 decoder가 있는 text-to-text 모델이다. | 질의를 만드는 학습 대상 |

IterCQR은 MS MARCO로 학습된 ANCE를 가져와 고정한다(§3.3). ANCE의 원래 negative 학습을 새로 수행하는 설정은 아니다.

선수 자료: [BM25 저자 교재](https://nlp.stanford.edu/IR-book/html/htmledition/okapi-bm25-a-non-binary-model-1.html), [ANCE 원 논문](https://arxiv.org/abs/2007.00808), [T5 원 논문](https://www.jmlr.org/papers/v21/20-074.html).

### 생성 확률과 검색 점수는 서로 다르다

AI 해설용으로, T5는 후보 문자열 $c=(w_1,\ldots,w_L)$에 다음 확률을 줄 수 있다.

$$
p_\theta(c\mid x)=\prod_{\ell=1}^{L}p_\theta(w_\ell\mid w_{<\ell},x),
\qquad
\log p_\theta(c\mid x)=\sum_{\ell=1}^{L}\log p_\theta(w_\ell\mid w_{<\ell},x).
$$

여기서 $x=(q,H)$이다. 이 확률은 **그 문장을 생성하기 쉬운가**, 검색 보상은 **정답 passage와 잘 맞는가**다. 긴 문장의 raw 확률은 토큰 확률 곱 때문에 작아질 수 있다. 원문의 후보 확률에 길이 보정이 정확히 어떻게 적용되는지는 본문만으로 미확인이므로 코드 대조 없이 구현을 단정하지 않는다.

## 3. 핵심 방법 / Method

### 무엇을 고정하고 무엇을 갱신하는가

Fig.2·Algorithm1에서 이전 모델이 turn별 후보를 만들고, ANCE가 gold passage와 비교해 보상을 준다. 해당 반복에서는 후보를 고정해 T5를 학습하고 다음 반복에서 재생성한다.

$c_i^j,p_i$는 후보·정답 passage 문자열이고 $C_i^j,P_i\in\mathbb R^d$는 각각의 벡터다. 구체적 $d$는 이 노트에서 미확인이다.

### Eq.1: 한 후보의 정답 여부 대신 후보 집합의 기대 비용

원문 §3.2.2 Eq.1을 기호를 정리해 쓰면 다음과 같다.

$$
\min_\theta\sum_{i=1}^{N}
\mathbb{E}_{\tilde p_\theta(y_i\mid x_i)}
[\mathcal C(x_i,y_i)].
\tag{원문 1}
$$

전체 문자열 대신 beam search 후보들로 기대값을 근사한다. $\tilde p$는 **후보들 안에서 재정규화한 생성 확률**이다. 보상을 softmax해 만드는 분포가 아니다.

다음 표기는 AI가 재정규화의 의미를 풀어 쓴 것이다. 후보 집합을 고정하고 raw sequence 확률을 사용하는 경우,

$$
\tilde p_j=\frac{p_\theta(c^j\mid x)}
{\sum_{m=1}^{n}p_\theta(c^m\mid x)}
=\operatorname{softmax}_j(s),\qquad s_j=\log p_\theta(c^j\mid x).
$$

합은 1이다. log 공간 계산과 log 확률 자체를 가중 평균하는 것은 다르다. 길이 보정·수치 안정화의 코드 구현은 미확인이다.

### Eq.2–3: 정답 passage와의 코사인 유사도를 보상으로

비용을 낮추는 대신 보상을 높이려면 음의 기대 보상을 최소화한다. 원문 Eq.2의 핵심은 다음이다.

$$
\mathcal L_{\mathrm{MBR}}
=-\sum_{i=1}^{N}\sum_{j=1}^{n}
\tilde p_\theta(c_i^j\mid q_i,H_{k-1})\,r_i^j,
\qquad
r_i^j=\mathcal R(C_i^j,P_i).
\tag{원문 2}
$$

$$
\mathcal R(C_i^j,P_i)
=\cos(C_i^j,P_i)
=\frac{(C_i^j)^\top P_i}{\|C_i^j\|\|P_i\|}.
\tag{원문 3의 정의를 전개}
$$

원문은 **생성 확률 재정규화와 별도로**, 좁게 몰린 보상을 min-max 정규화해 $[0,1]$ 범위를 사용한다고 적는다. 원래 cosine의 범위 $[-1,1]$와 정규화 후 보상 범위를 혼동하지 말자. 질의별·배치별 중 어느 범위로 min/max를 잡는지, 모든 보상이 같을 때의 epsilon 처리는 본문에서 명확히 확인하지 못했으며 코드 검증이 필요하다.

여기서 보상은 실수값 cosine이지만 **MRR이나 BM25 순위 자체는 아니다**. 다른 문서가 얼마나 높은 점수를 받는지도 순위에 영향을 준다. 관련성 대리신호를 좋아지게 하는 일이 실제 검색기 지표로 전이되는지는 실험으로 확인해야 한다.

### 보상을 미분하지 않는데 생성기가 왜 바뀌는가

아래는 **AI의 수학적 유도**이며 원문에 실린 별도 식이 아니다. 한 입력의 후보 집합과 보상 $r_j$를 고정하고 $\bar r=\sum_j\tilde p_jr_j$, $L=-\bar r$로 두면,

$$
\frac{\partial L}{\partial s_j}
=-\tilde p_j(r_j-\bar r).
\tag{AI 유도}
$$

이는 softmax 미분 $\partial\tilde p_m/\partial s_j
=\tilde p_m(\mathbf1_{m=j}-\tilde p_j)$을 대입한 결과다. 평균보다 보상이 큰 후보의 log-score에는 음의 gradient가 생겨 gradient descent에서 그 점수를 올리는 방향으로 작용한다. 실제 T5에서는 후보 점수가 공유 파라미터에 의존하므로 각 점수를 완전히 독립적으로 조절할 수 있는 것은 아니다.

예를 들어 $\tilde p=(0.5,0.3,0.2)$, $r=(1,0.5,0)$라면 기대 보상은 0.65, 손실은 −0.65, 점수 gradient는 $(-0.175,0.045,0.13)$이다. 최고 보상 후보 쪽 확률 질량을 늘리는 직관을 보여준다. 후보를 문자열로 생성하는 이산 과정을 그대로 역전파하는 것이 아니라, 고정된 후보 문자열의 조건부 생성 확률을 미분한다. 검색 인코더는 고정되어도 학습 신호는 존재한다.

### τ=1은 단계 경계이며 epoch 수가 아니다

| 단계 | 후보·target | 손실 | 원문 근거 |
|---|---|---|---|
| M0 초기화 | gpt-3.5-turbo 재작성 | NLL | §3.2.1 |
| M1, $t\le\tau=1$ | M0가 만든 후보 10개 | MBR | §3.2.2, §4 |
| M2 이후 | 이전 모델 후보 중 **보상 최고** 1개 | NLL | §3.2.3 |

Top-1은 생성 확률 최고 후보와 다를 수 있다. $c_i^{top}=\arg\max_j r_i^j$를 정답 문자열로 삼는 것이며, 원문 Eq.4는 다음이다.

$$
\mathcal L_{\mathrm{NLL}}
=-\sum_i\log p_\theta(c_i^{top}\mid q_i,H_{k-1}).
\tag{원문 4}
$$

NLL은 target의 각 토큰 log 확률을 더한 값을 높인다. MBR은 여러 후보의 상대적 보상을 활용하고 Top-1 NLL은 선택한 문자열을 집중적으로 학습한다. M1은 한 번의 optimizer step이나 한 epoch가 아니다. Appendix A에서 MBR은 2 epochs이다.

저자들은 초기 후보 품질 다양성에는 MBR이 유리하지만, 좋아진 후보들 사이의 작은 상대 차이를 계속 벌주면 불안정해질 수 있다고 해석한다(§4.4, Fig.3). 이는 관측의 해석이다.

### supervision과 추론 비용의 경계

사람 rewrite는 없어도 **정답 passage·대화 질문/답변·초기 생성 데이터는 필요**하다. gold passage 없는 도메인 적용은 별도 문제다. 학습용 정답과 테스트 추론 입력을 구분하자. 원문 추론은 대화→최종 CQR→고정 검색기다(§3.1–3.3).

추론 때 LLM 재작성이나 후보 10개를 평가하는 전체 반복 훈련은 필요 없다. 실제 배포 latency와 공개 모델 구동 결과는 측정하지 않았다.

## 4. 실험과 근거 / Evidence

### 지표가 각각 묻는 것

원문 §4는 pytrec_eval로 MRR, nDCG@3, Recall@10/100을 계산한다. 아래 계산은 **AI의 교육용 예**이며 데이터셋의 모든 qrel 구조를 단순화한 설명이다.

- MRR은 첫 관련 문서 순위의 역수를 질의마다 평균한다. 세 질의에서 첫 관련 문서가 1위, 4위, 평가 목록 밖이면 $(1+1/4+0)/3=0.4167$이다.
- Recall@K는 정답 관련 문서 중 상위 K에 포함된 비율이다. 정답 passage가 하나일 때는 top-K 적중 여부와 같다. 여러 정답이면 단순 hit와 다르다.
- nDCG@3는 상위 세 위치의 관련성 이득을 순위에 따라 할인하고 이상적 순위로 나눈다. 이진 정답 하나가 3위에 있는 toy 예는 $1/\log_2(4)=0.5$이며 1위면 1이다.

MRR이 올라가면서 R@100이 떨어질 수 있다. 이미 찾는 질의에서는 정답을 위로 끌어올리지만, 일부 어려운 질의에서는 100위 안에도 못 찾게 될 수 있기 때문이다. 최종 답변기가 상위 5개만 읽는지 100개를 rerank하는지에 따라 중요한 지표도 달라진다는 것이 AI의 시스템 관점이다.

### Table1: 정확한 보고값과 비교 조건

아래는 원문 Table1(p.8126)의 일부 행이다. MRR·nDCG는 소수, recall은 표의 백분율 단위를 유지했다. 아래 값은 저자 보고이며 사용자가 재현한 결과가 아니다.

| 데이터·검색기 | 방법 | MRR | nDCG@3 | R@10 | R@100 |
|---|---|---:|---:|---:|---:|
| TopiOCQA·ANCE | Initial | 0.178 | 0.168 | 32.6 | 47.7 |
| TopiOCQA·ANCE | ConvGQR | 0.256 | 0.243 | 41.8 | 58.8 |
| TopiOCQA·ANCE | IterCQR | 0.263 | 0.251 | 42.6 | 62.0 |
| TopiOCQA·BM25 | ConvGQR | 0.124 | 0.107 | 23.8 | 45.6 |
| TopiOCQA·BM25 | IterCQR | 0.165 | 0.149 | 29.3 | 54.1 |
| QReCC·dense* | CONQRR | 0.418 | 미보고 | 65.1 | 84.7 |
| QReCC·ANCE | ConvGQR | 0.420 | 0.391 | 63.5 | 81.8 |
| QReCC·ANCE | IterCQR | 0.429 | 0.402 | 65.5 | 84.1 |
| QReCC·BM25 | CONQRR | 0.383 | 미보고 | 60.1 | 88.9 |
| QReCC·BM25 | ConvGQR | 0.441 | 0.410 | 64.4 | 88.0 |
| QReCC·BM25 | IterCQR | 0.467 | 0.441 | 64.4 | 85.5 |

*CONQRR dense는 DualEncoder이며 ANCE가 아니다. 서로 다른 dense retriever 결과를 질의 생성기만의 통제 실험으로 해석하지 않는다. §4 Baselines는 여러 기준선 결과를 각 선행논문에서 가져왔음을 밝힌다. 모든 baseline을 같은 코드·seed로 재실행한 비교라고 적을 근거는 없다.

TopiOCQA BM25 MRR 0.124→0.165는 절대 +0.041, 상대 약 +33.1%다. 반면 ANCE는 0.256→0.263, 절대 +0.007이다. “33.1%p 상승”은 잘못된 단위다. QReCC BM25 R@100은 ConvGQR보다 2.5%p 낮다. 모든 지표에서 최고라는 결론은 Table1이 지지하지 않는다.

### 절제실험과 일반화에서 확인된 범위

Table4(p.8127)의 TopiOCQA 절제실험 MRR은 다음이다.

| 방법 | ANCE MRR | BM25 MRR |
|---|---:|---:|
| IterCQR | 0.263 | 0.165 |
| OnlyMBR | 0.216 | 0.111 |
| OnlyTop1 | 0.248 | 0.150 |

이 설정에서 두 단계 조합이 유리하다. τ=1의 근거는 §4.4·Fig.3이다. 넓은 τ 탐색표, 후보 수 3·5, 다른 보상 정규화의 효과는 이 표로 알 수 없다.

Table2에서 TopiOCQA로 학습→QReCC 평가인 OOD IterCQR의 BM25 MRR은 0.449, QReCC ID ConvGQR은 0.441이다. 그러나 같은 OOD IterCQR의 dense MRR 0.401은 ID ConvGQR 0.420보다 낮다. “도메인이 바뀌어도 항상 우수하다”보다는 이 두 데이터 간 일부 조건에서 전이가 관측되었다고 읽는다.

Table3에서 TopiOCQA 데이터 50% 사용 시 ANCE MRR은 0.252, 100%는 0.263이다. 데이터 절반으로 정확히 같은 성능이라고 쓰지 않는다. §5 Fig.5는 질의 길이·대화 및 gold passage와의 token overlap 증가를 보여준다. 이를 저자는 요약 확장 학습으로 해석하지만, 별도의 사람 평가로 요약의 충실성을 증명했다는 뜻은 아니다.

§6 Table5는 같은 M0에서 출발한 M1의 MBR/SCST 비교다. SCST에는 sampling, MBR에는 beam search를 썼으므로 목적함수 외 후보 생성 방식도 함께 다르다. 이 표를 최종 IterCQR 대 원래 CONQRR의 동일 조건 재현으로 취급하지 않는다.

### 자원·불확실성도 실험 결과의 일부다

§4 Implementation Details는 T5-base, 후보 10, τ=1, Adam, learning rate $10^{-5}$, batch 8, maximum query length 32를 보고한다. Appendix A는 trainable parameters 220M, 단일 Nvidia A6000, 단일 run을 명시한다. 이 노트에서 tokenizer 길이 계산과 상세 seed·라이브러리 버전은 미확인이다.

| Appendix A의 보고 | TopiOCQA | QReCC |
|---|---|---|
| M0 epochs | 5 | 10 |
| MBR epochs | 2 | 2 |
| Top-1 epochs | 5 | 5 |
| 반복 횟수 표현 | 15 iterations | 5 iterations |

보고 시간은 MBR 약 5시간/epoch, Top-1 약 40분/epoch이다. 사용자 GPU 시간, 후보 생성·색인·LLM API를 포함한 총비용은 미확인이다. 단일 run이므로 seed 분산·신뢰구간은 없다. split별 건수·별도 누수 감사는 미확인이다.

## 5. 한계·비판·질문 / Limits & questions

**저자가 명시한 한계(Limitations):** 초기 LLM 추론 비용과 품질 의존, 후보 저장 공간, 반복 학습 시간, dense 보상이 sparse 성능 개선을 항상 보장하지 않는 점이다. Ethical Statement는 영어 공개 데이터만 사용했고, 대화 이력의 편향·부적절한 내용이 질의 확장으로 전달될 가능성을 적는다.

**AI 분석 질문:** cosine과 순위의 불일치, 길이 증가 효과, topic shift에서 불필요한 과거 정보를 덧붙이는 문제를 분리해 볼 가치가 있다. 이는 저자가 검증한 새 결과가 아니다.

**사용자가 직접 남길 칸 — 미작성:** 내가 이해하지 못한 식 / 논문 주장에 동의하지 않는 이유 / 기대와 달랐던 오류 예 / 실제로 배운 점. 이 문단의 빈칸은 학습을 끝냈다는 기록이 아니다.

## 6. 인턴십 연구로 확장 / Research seed

다음은 **AI 제안·미검증, 신규성 미확인**이다. 후보 수 10을 줄여도 같은 계산 예산에서 유사한 검색 성능을 얻을 수 있는가? 단순히 후보가 적으면 학습이 빠르다는 관찰을 넘어, 후보 다양성·반복 수·전환 시점의 어떤 조합이 효율적인지 좁혀야 연구 질문이 된다.

가설은 후보 5개로 생성 비용을 절약하고 남은 예산을 반복에 쓰면 MRR 손실을 줄일 수 있다는 것이다. 기각 조건은 같은 시간에서 기준선보다 낮은 MRR이 반복적으로 나오거나, 계산 절약이 실제로 관측되지 않는 경우다. 성능이 유지될 것이라고 미리 결론 내리지 않는다.

최소 실험은 공개 M0 또는 동일 초기 데이터, 같은 train/dev/test 분리, 같은 검색기·최대 길이·τ에서 후보 수 하나만 바꾼다. 주 지표 MRR과 R@100, 생성·학습 wall time, 후보 중복률을 함께 기록한다. 고정 반복 수 비교와 고정 시간 비교는 서로 다른 질문이므로 결과표를 구분한다. 여러 seed가 불가능하면 단일 run의 제약을 명시한다.

3개월 후보 계획은 1개월차 코드·데이터·논문 baseline 실행 및 최근 효율화 연구 검토, 2개월차 후보 수 변경 하나와 통제 비교, 3개월차 오류 분석·반복 검증·보고다. 장비가 부족하면 공개 checkpoint의 추론·후보 다양성 분석으로 범위를 줄인다. 사용자 GPU·저장 공간·코드 호환성·데이터 라이선스는 미확인이다. 일정 확정·실험 실행 기록은 아니다.

## 7. 연결할 읽을거리와 검증 / Follow-up

| 자료 | 읽을 지점·관계 |
|---|---|
| [CONQRR (Wu et al., EMNLP 2022)](https://aclanthology.org/2022.emnlp-main.679/) | IterCQR §2.1의 검색 보상·SCST 비교. CONQRR 원문 §3.2는 black-box 검색기 일반 지원을 목표로 하며 BM25와 DualEncoder를 평가한다. IterCQR의 BM25 이진 보상 대조를 CONQRR 전체가 BM25 전용이라는 말로 확대하지 않는다. 원문에는 zero rewrite supervision 실험도 있다. |
| [ConvGQR (Mo et al., ACL 2023)](https://aclanthology.org/2023.acl-long.274/) | 재작성+예상 답변 확장과 강한 Table1 비교 대상. IterCQR Appendix C Table7에서 이 확장을 덧붙일 여지가 보고된다. |
| [Minimum Risk Annealing (Smith & Eisner, 2006)](https://aclanthology.org/P06-2101/) | IterCQR이 직접 인용하는 기대 손실 최적화의 기반. 원 논문 §2.1 식(3)을 먼저 읽는다. annealing 전체를 IterCQR이 그대로 사용한다는 뜻은 아니다. |

AI 추천 읽기 순서는 Fig.1→Fig.2·Algorithm1→Table1→§3.2 수식→§4.4이다. [핵심 노트](./README.md), [쉬운 설명](./easy-explanation.md), [퀴즈](./quiz.md), [선행논문 해설](./references.md)를 함께 사용한다.

AI 확인 범위는 출판 서지, 본문·부록 수치, 핵심 수식, 공식 저장소 여부다. 소스코드 내부 정규화와 실행 결과는 검증하지 않았다. 검증할 다음 항목은 후보 확률의 길이 보정, min-max 축·동률 처리, 정확한 checkpoint/iteration 대응, seed와 평가 qrel 처리이다.

출처 표시: Jang et al. (2024), Association for Computational Linguistics, DOI [10.18653/v1/2024.naacl-long.449](https://doi.org/10.18653/v1/2024.naacl-long.449). [ACL 안내](https://aclanthology.org/2024.naacl-long.449/)에 따라 2016년 이후 출판 자료는 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)이다. 이 문서는 원문을 인용·재구성한 한국어 교육용 해설이며, AI 유도·예·가설은 원 저자의 추가 결과가 아니다.
