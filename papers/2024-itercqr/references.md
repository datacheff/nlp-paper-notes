# IterCQR를 이해하기 위한 핵심 참고논문

[IterCQR 학습 안내](README.md) · [상세 설명](detailed-explanation.md) · [전체 읽기 목록](../../reading-list.md)

이 문서는 IterCQR가 실제로 인용한 **CONQRR, ConvGQR, Smith & Eisner (2006)**의 핵심 읽기 카드다. 세 원문에서 관련 방법을 직접 확인해 작성했다. 사용자의 원문 정독·구현 상태는 모두 **미시작·미시도**이며, 각 논문의 완전한 정독 자료는 앞으로 한 편씩 확장한다. 예시·이해 질문은 학습용으로 새로 만들었으며 원논문의 실험 결과가 아니다.

MILAB 대상 논문의 선별 범위는 2022~2026년이지만, 설명에 필요한 참고논문에는 그 제한을 적용하지 않는다. 쪽수는 PDF 뷰어의 페이지 순번 대신 **논문에 인쇄된 쪽수**를 적었다.

## 먼저 구분할 질문

| 읽을 논문 | IterCQR를 읽다가 막히는 질문 | 먼저 볼 원문 위치 |
| --- | --- | --- |
| CONQRR | 사람 rewrite를 흉내 내는 대신 검색 보상을 학습에 사용할 수 있는가? | §3.2, Eq. (2)–(3), §4.3 Figure 3 |
| ConvGQR | 질문을 명확하게 고치는 것과 검색용 정보를 덧붙이는 것은 어떻게 다른가? | §3.2.1–3.3, Figure 2, Eq. (3)–(5) |
| Minimum Risk Annealing | 후보마다 점수가 있을 때 왜 최고 후보 하나만 정답으로 쓰지 않는가? | §2.1 Eq. (3), §3 Eq. (5)–(6) |

## 1. CONQRR — 검색 결과로 rewrite를 학습하기

**서지:** Zeqiu Wu et al., 2022, *CONQRR: Conversational Query Rewriting for Retrieval with Reinforcement Learning*, EMNLP, pp. 10000–10014. [공식 페이지](https://aclanthology.org/2022.emnlp-main.679/) · [PDF](https://aclanthology.org/2022.emnlp-main.679.pdf)

**문제·입출력.** 대화 이력과 현재 질문을 입력받아 독립적인 검색 질의를 출력한다. 자연스러운 사람 rewrite가 검색에 최적이라는 보장이 없어, retriever를 고정하고 재작성기를 검색 보상으로 학습한다.

**핵심 원리.** T5가 sampled rewrite와 greedy rewrite를 만든다. in-batch 문서 후보에서 positive passage를 1위에 놓으면 `score=1`, 아니면 `0`이다. SCST는 두 rewrite의 점수 차를 reward로 삼는다. 사람 rewrite를 쓰는 혼합 학습 외에 **zero rewrite supervision**도 실험했다. BM25 전용 방법은 아니며 black-box retriever를 대상으로 설계되었다. [근거: §3.2, pp. 10002–10004, Eq. (2)–(3); §4.3, Figure 3](https://aclanthology.org/2022.emnlp-main.679.pdf#page=3)

**IterCQR와 차이·한계.** IterCQR는 이전 iteration의 고정 후보를 학습하며, gold passage와의 연속적인 cosine reward를 쓴다. CONQRR의 이진 in-batch 성공은 전체 corpus의 MRR와 다르다. 따라서 “IterCQR가 최초로 사람 rewrite를 제거했다”는 해석은 피한다. [대조 근거: IterCQR §2.1, pp. 8122–8123](https://aclanthology.org/2024.naacl-long.449.pdf#page=2)

**미니 이해 질문.** sampled rewrite와 greedy rewrite가 모두 positive passage를 1위로 찾았다. sampled 쪽의 문장이 더 자연스러우면 이 보상에서 더 유리할까?

<details>
<summary>해설 보기</summary>

아니다. 이 예에서 reward는 `1 − 1 = 0`이다. 자연스러움은 해당 점수에 들어 있지 않다. 이는 학습용 계산이며, 문장 품질과 검색 품질을 별도로 측정해야 하는 이유다. 반대로 둘 다 실패해도 차는 0이므로, 두 실패 후보의 미세한 차이를 이 이진 신호만으로 구분하지 못한다.

</details>

## 2. ConvGQR — rewrite와 예상 답변을 합쳐 검색하기

**서지:** Fengran Mo et al., 2023, *ConvGQR: Generative Query Reformulation for Conversational Search*, ACL, pp. 4998–5012. [공식 페이지](https://aclanthology.org/2023.acl-long.274/) · [PDF](https://aclanthology.org/2023.acl-long.274.pdf)

**문제·입출력.** 대화 이력·현재 질문에서 검색 질의를 만든다. 명확한 질문에도 관련 문서를 찾을 단서가 부족할 수 있어, 두 생성기가 각각 **rewrite**와 **potential answer**를 출력하고 이어 붙인다.

**핵심 원리.** rewrite는 사람 재작성, 답변 생성기는 gold answer를 목표로 학습한다. 추가로 frozen retriever의 relevant passage 표현에 생성기 표현을 가깝게 하는 MSE를 사용한다. 이 knowledge infusion은 학습 단계에만 적용한다. [근거: §3.2.1–3.3, pp. 5000–5002, Eq. (3)–(5)](https://aclanthology.org/2023.acl-long.274.pdf#page=3)

**IterCQR와 차이·한계.** ConvGQR는 두 생성기를 따로 학습하며, 예상 답변의 정확성은 보장되지 않는다. IterCQR는 한 재작성기를 후보 reward로 반복 개선한다. 다만 두 아이디어는 결합 가능하다. IterCQR Appendix C의 Table 7에서 답변 확장을 더한 MRR는 TopiOCQA `0.263→0.277`, QReCC `0.429→0.444`였다. [비교: IterCQR Table 1 및 Appendix C](https://aclanthology.org/2024.naacl-long.449.pdf#page=13)

**미니 이해 질문.** 질문 “그 도시는 언제 세워졌어?”를 완전한 질문으로 고쳤다. 이어 붙일 예상 답변이 틀리면 항상 검색도 실패할까?

<details>
<summary>해설 보기</summary>

항상 그렇지는 않다. 예를 들어 잘못 예상한 연도가 실제 연도와 함께 비교·정정되는 문서를 찾는 단서가 될 수도 있다. 반대로 엉뚱한 도시의 연도는 검색을 방해할 수 있다. 따라서 생성한 답의 정확도와 그 답을 넣은 질의의 검색 성능은 서로 다른 측정 대상이다. 이 예는 원리를 설명하기 위한 가상 상황이다.

</details>

## 3. Minimum Risk Annealing — 여러 후보의 기대손실 줄이기

**서지:** David A. Smith and Jason Eisner, 2006, *Minimum Risk Annealing for Training Log-Linear Models*, COLING/ACL Poster Sessions, pp. 787–794. [공식 페이지](https://aclanthology.org/P06-2101/) · [PDF](https://aclanthology.org/P06-2101.pdf)

**문제·입출력.** 입력별 후보 구조·특징·손실을 받아 log-linear 모델의 가중치를 학습한다. 번역·구문분석처럼 출력이 이산적이면 1-best 손실은 가중치의 작은 변화에 반응하지 않다가 갑자기 바뀐다.

**핵심 원리.** §2.1 Eq. (3)은 후보 확률로 가중한 기대손실을 최소화한다. §3에서는 분포를 점차 날카롭게 하는 annealing을 다룬다. 기대손실은 미분 가능해지지만 전역 최적점은 보장하지 않는다. [근거: pp. 788–789, Eq. (2)–(6)](https://aclanthology.org/P06-2101.pdf#page=2)

```math
\mathcal{R}(\theta)=\sum_i\sum_y p_\theta(y\mid x_i)L_i(y)
```

**IterCQR와 차이·한계.** IterCQR는 이 원리를 beam 후보의 재정규화 확률과 음의 검색 reward에 적용한다. 원논문의 log-linear annealing 전체를 그대로 구현한 것은 아니다. IterCQR의 `τ`는 MBR에서 Top-1 학습으로 바꾸는 iteration 경계다. 후보 집합에 좋은 질의가 없으면 이 목적함수만으로 그것을 보장할 수 없다. [적용 근거: IterCQR §3.2.2, Eq. (1)–(3)](https://aclanthology.org/2024.naacl-long.449.pdf#page=4)

**미니 이해 질문.** 후보 A·B의 reward가 `0.2, 0.8`이고 확률이 `0.75, 0.25`다. 확률을 `0.5, 0.5`로 바꾸면 기대 reward는?

<details>
<summary>해설 보기</summary>

처음은 `0.75×0.2 + 0.25×0.8 = 0.35`, 변경 후는 `0.5×0.2 + 0.5×0.8 = 0.50`이다. reward가 높은 B로 확률을 옮기면 음의 기대 reward는 작아진다. 1-best가 무엇인지 확인하는 것과 후보 전체의 확률 질량을 학습하는 것이 다름을 보여 주는 가상 예다. IterCQR 실험 수치가 아니다.

</details>

## 세 논문을 연결해 말해 보기

학습 후에는 아래 세 문장을 원문을 보지 않고 자기 말로 설명한다.

1. CONQRR를 보면 **검색을 잘하는 rewrite**를 학습 목표로 삼는 배경을 이해할 수 있다. 사람 재작성과의 유사도만이 목표는 아니다.
2. ConvGQR를 보면 **질문을 완전하게 만드는 일**과 **검색 단서를 확장하는 일**을 구분할 수 있다. 예상 답변 확장은 IterCQR와 결합해 검토할 요소다.
3. 기대손실 관점에서는 **좋은 후보 하나를 고르는 것**에 앞서 **후보들의 확률을 어떻게 이동시킬지**를 생각할 수 있다. 원논문의 annealing과 IterCQR의 학습 단계 전환은 따로 설명한다.

다음 정독 때에는 CONQRR §3.2·§4.3, ConvGQR §3.2·ablation, Smith & Eisner §2·§3을 먼저 읽고 실험·한계까지 확장한다. 지금의 카드는 이 연결을 잡는 자료이며, 세 논문의 모든 실험을 검증한 재현 기록은 아니다.
