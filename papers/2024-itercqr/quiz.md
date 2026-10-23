# IterCQR 사전·사후 퀴즈

[논문 노트](README.md) · [쉬운 설명](easy-explanation.md) · [상세 설명](detailed-explanation.md) · [핵심 참고문헌](references.md) · [공식 원문](https://aclanthology.org/2024.naacl-long.449.pdf)

**사전:** 정답을 펼치지 말고 질문을 읽어 아는 만큼 쓴다. 계산식·이유·모르는 점을 남긴다. **사후:** 원문과 설명을 읽은 뒤 새 답을 작성하고, 그다음 문항별 정답을 펼친다. 사전 답을 덮어쓰지 않으면 이해의 변화를 비교하기 쉽다. 아래 가상 대화·계산용 숫자·연구 설계 예시는 논문의 실행 결과가 아니다.

총 14문항, 100점이다. 계산·이유·조건까지 써야 만점이다. 정답 암기나 100점만으로 완벽한 이해·구현 능력을 판정하지 않는다. 마지막에는 정답을 덮고 새 예시로 설명하고, 필요하면 코드와 실제 검색 결과로 확인한다.

## 1. 모델의 역할 — 5점

IterCQR의 입력과 직접 출력은 무엇인가? 최종 답변까지 생성한다고 설명하면 무엇이 잘못인가?

<details>
<summary>정답·이유·채점 기준</summary>

입력은 현재 질문과 이전 질문·답변의 맥락, 직접 출력은 독립적으로 검색 가능한 재작성 질의다. 이어지는 검색기는 관련 문단의 순위 목록을 반환한다. 사용자 답변 생성은 별도 단계이며 본 논문의 검색 지표와 구분해야 한다.

배점: 입력 2점, 재작성 질의 2점, 답변 생성과의 구분 1점. 다시 읽기: [원문 §3.1](https://aclanthology.org/2024.naacl-long.449.pdf#page=3), [Fig. 1](https://aclanthology.org/2024.naacl-long.449.pdf#page=1), [쉬운 설명](easy-explanation.md).

</details>

## 2. 생략을 푸는 작은 예시 — 5점

**가상 대화:** “은하박물관은 어디에 있어?” → “가상도시 북쪽에 있어.” → “거기 주차장은 있어?” 마지막 질문의 rewrite를 하나 쓰고, 질문을 그대로 검색할 때 빠지는 정보를 말하라. 주차장 유무를 새로 지어내지 말 것.

<details>
<summary>가능한 답·이유·채점 기준</summary>

예: “은하박물관에 주차장이 있는가?” `거기`의 지시 대상을 복원한다. “은하박물관에는 주차장이 있다”는 문장은 근거 없이 답을 추가하므로 적절한 rewrite 예가 아니다. 실제 후보의 검색 성능은 실행 전에는 모른다.

배점: 대상 복원 2점, 질문 의도 유지 2점, 근거 없는 답을 넣지 않음 1점. 다시 읽기: [원문 §3.1](https://aclanthology.org/2024.naacl-long.449.pdf#page=3), [쉬운 설명](easy-explanation.md).

</details>

## 3. 두 검색기의 차이 — 5점

BM25와 ANCE가 질의·문단의 관련성을 표현하는 방식을 비교하라. dense 보상이 높아지면 BM25 검색 결과도 반드시 좋아지는가?

<details>
<summary>정답·이유·채점 기준</summary>

BM25는 단어 일치·빈도·문서 길이 등을 활용하는 sparse 검색 방식, ANCE는 질의와 문단의 dense 표현을 사용하는 방식이다. dense cosine 보상을 개선해도 BM25의 순위 개선이 보장되지는 않는다. 모델이 최적화한 점수와 실제 검색기가 사용하는 점수가 다르기 때문이다.

배점: 두 방식의 차이 3점, 보장되지 않는 이유 2점. 다시 읽기: [원문 §3.3](https://aclanthology.org/2024.naacl-long.449.pdf#page=5), [Limitations](https://aclanthology.org/2024.naacl-long.449.pdf#page=9), [상세 설명](detailed-explanation.md).

</details>

## 4. 초기 LLM의 역할 — 5점

초기 `gpt-3.5-turbo` 출력은 어디에 쓰는가? 이후 모든 학습 후보를 계속 같은 LLM이 만드는가? 추론 때도 LLM 초기 데이터셋 생성이 필요한가?

<details>
<summary>정답·이유·채점 기준</summary>

초기 rewrite 데이터셋 `D₀`를 만들고, T5-base 초기 모델 `M₀`가 이를 NLL로 학습한다. 이후 후보는 이전 반복의 재작성 모델이 생성한다. 추론에서는 최종 재작성 모델을 사용하므로 초기 데이터셋을 다시 생성하지 않는다.

배점: 초기 pseudo-label 역할 2점, 이후 생성 주체 2점, 추론 구분 1점. 다시 읽기: [원문 §3.2.1, Algorithm 1](https://aclanthology.org/2024.naacl-long.449.pdf#page=4), [§1](https://aclanthology.org/2024.naacl-long.449.pdf#page=2), [상세 설명](detailed-explanation.md).

</details>

## 5. 전체 학습 흐름 — 10점

`τ=1`, 후보 수 10일 때 `M₀`, `M₁`, `M₂` 학습을 순서대로 설명하라. 각 단계의 후보 생성 주체, 쓰는 loss, Top-1의 선택 기준을 포함하라. `M₁` 학습 도중 매 gradient step에 후보를 새로 생성하는 구조로 설명해도 되는가?

<details>
<summary>정답·이유·채점 기준</summary>

`M₀`: 초기 LLM rewrite를 NLL로 학습. `M₁`: `M₀`가 생성한 입력별 10개 후보로 MBR 학습. `M₂`: `M₁`이 생성한 후보 중 gold passage와의 cosine 보상이 최고인 후보를 목표로 NLL 학습한다. 이후도 `t>1`이면 Top-1 NLL 단계다. 생성 확률이 최고인 후보를 고른다는 설명은 틀리다.

Algorithm 1은 반복마다 이전 모델로 후보 데이터셋을 준비한 다음 학습하는 구조다. 매 gradient step마다 후보를 새로 생성하는 설명과 구분한다. 후보 텍스트가 고정되어도 현재 모델의 후보 확률은 업데이트될 수 있다.

배점: M₀ 2점, M₁ 2점, M₂·τ 2점, 보상 기반 선택 2점, 후보 고정과 확률 갱신 구분 2점. 다시 읽기: [원문 Algorithm 1, §3.2.2–3.2.3](https://aclanthology.org/2024.naacl-long.449.pdf#page=4), [§4 구현 조건](https://aclanthology.org/2024.naacl-long.449.pdf#page=6), [상세 설명](detailed-explanation.md).

</details>

## 6. 무엇이 없어도 되고, 무엇이 필요한가? — 10점

“사람 rewrite를 안 쓰므로 이 방법은 정답 주석도 LLM 비용도 필요 없는 완전 무지도 학습이다.” 이 문장의 서로 다른 오류를 고쳐라. 새 테스트 질문에서 gold passage를 입력으로 주는지도 구분하라.

<details>
<summary>정답·이유·채점 기준</summary>

사람이 작성한 rewrite label 없이 학습하지만, 초기 LLM rewrite에는 생성 비용·모델 의존성이 있다. 보상 계산에는 학습 질문의 gold passage annotation을 사용한다. 따라서 무주석 학습이라는 결론은 성립하지 않는다. 추론 때 gold passage를 알려 주어 rewrite를 선택하는 구조도 아니다. 테스트의 정답 관련성 정보는 평가에 쓰며, 이를 추론 입력으로 사용하면 다른 설정이 된다.

배점: human rewrite label 구분 2점, gold passage supervision 3점, LLM 비용 2점, 학습·평가·추론 구분 3점. 다시 읽기: [원문 §2.1, §3.2](https://aclanthology.org/2024.naacl-long.449.pdf#page=3), [Limitations](https://aclanthology.org/2024.naacl-long.449.pdf#page=9), [쉬운 설명](easy-explanation.md).

</details>

## 7. 보상과 확률 구분 — 5점

Eq. (2)의 후보 확률과 Eq. (3)의 보상은 무엇이 다른가? 무엇이 고정되어 있고 무엇이 학습되는가? “reward를 softmax한 값으로 T5를 학습한다”는 요약은 정확한가?

<details>
<summary>정답·이유·채점 기준</summary>

후보 확률은 현재 재작성 모델이 각 후보 텍스트를 낼 가능성을 후보 집합에서 재정규화한 값이다. 보상은 고정된 dense encoder로 구한 후보·gold passage 표현의 cosine을 바탕으로 하며, 본문은 min-max 정규화를 설명한다. Eq. (2)는 후보 확률에 보상을 곱한 기대값을 사용한다. 보상 자체를 softmax한 확률과 동일하지 않다. 학습 대상은 재작성기이고 ANCE encoder는 고정이다.

배점: 확률과 보상 구분 2점, 고정 encoder·학습 재작성기 2점, softmax 오해 교정 1점. min-max의 정확한 집계 범위·epsilon·sequence length normalization은 본문만으로 확정하지 말고 코드 확인 항목으로 남긴다. 다시 읽기: [원문 §3.2.2, Eq. (2)–(3)](https://aclanthology.org/2024.naacl-long.449.pdf#page=4), [상세 설명](detailed-explanation.md).

</details>

## 8. MRR 계산 — 10점

**교육용 가정:** 질문 세 개의 첫 관련 문단 순위가 각각 1위, 4위, 평가 범위 안에 없음이다. 미검색 질문의 reciprocal rank는 0으로 둔다. MRR을 구하라. 모든 관련 문단을 얼마나 회수했는지까지 이 값만으로 알 수 있는가?

<details>
<summary>정답·이유·채점 기준</summary>

각 reciprocal rank는 `1`, `1/4`, `0`이다. `MRR=(1+0.25+0)/3=5/12≈0.4167`. 평균 순위를 먼저 구한 뒤 역수를 취하면 틀린다. MRR은 첫 관련 문단의 순위를 반영하므로, 이것만으로 모든 관련 문단의 회수 정도를 알 수 없다. 실제 평가에서는 공식 qrels·검색 깊이·평가 도구의 설정을 확인한다.

배점: 질문별 reciprocal rank 3점, 평균과 값 4점, recall과의 차이 3점. 수치는 직접 만든 계산 예시다. 다시 읽기: [원문 §4 Evaluation Metrics](https://aclanthology.org/2024.naacl-long.449.pdf#page=5), [상세 설명](detailed-explanation.md).

</details>

## 9. 후보 softmax와 기대 보상 계산 — 10점

**교육용 가정:** 후보 세 개의 모델 log-score가 `s=(ln 0.1, ln 0.2, ln 0.3)`, raw cosine 보상이 `r=(0.2, 0.5, 0.8)`이다. 이번 문제에서는 **이 세 후보 안에서** min-max 정규화한다고 가정한다.

1. `pⱼ=exp(sⱼ)/Σ exp(sₖ)`를 구하라.
2. `r′ⱼ=(rⱼ−min r)/(max r−min r)`를 구하라.
3. 기대 보상 `Σpⱼr′ⱼ`와 한 입력의 MBR loss `−Σpⱼr′ⱼ`를 구하라.
4. 이 수치를 논문의 정확한 코드 동작이라고 단정할 수 있는가?

<details>
<summary>정답·이유·채점 기준</summary>

`exp(s)=(0.1,0.2,0.3)`이고 합은 `0.6`이므로 `p=(1/6,1/3,1/2)`. 정규화한 보상은 `r′=(0,1/2,1)`. 기대 보상은 `0+1/6+1/2=2/3`, loss는 `−2/3`이다. 보상의 softmax를 계산한 것이 아니다. 후보 log-score를 softmax하여 후보 확률을 만들었다.

이는 Eq. (2)를 이해하려는 단순화다. 원문은 beam 후보 확률의 재정규화와 reward min-max를 설명하지만, 실제 sequence scoring·길이 처리·정규화 범위 등의 구현까지 이 예시가 증명하지 않는다. 모든 reward가 같으면 min-max 분모가 0이므로 코드의 예외 처리도 확인해야 한다.

배점: p 3점, r′ 2점, 기대 보상·loss 3점, 가정과 코드 미확인 구분 2점. 다시 읽기: [원문 §3.2.2, Eq. (1)–(3)](https://aclanthology.org/2024.naacl-long.449.pdf#page=4), [상세 설명](detailed-explanation.md).

</details>

## 10. 고정 보상인데 gradient는 어디서 나오나? — 10점

**교육용 유도:** 후보와 보상을 고정하고 `p=softmax(s)`, `L=−Σpⱼrⱼ`라 하자. `∂L/∂sⱼ`를 유도하라. 두 후보에서 `p=(0.4,0.6)`, `r=(0.2,0.8)`일 때 각 gradient와 gradient descent 후 score의 변화 방향을 구하라. 이 학습에 beam search나 검색 순위 연산의 미분이 필요한가?

<details>
<summary>유도·정답·채점 기준</summary>

`∂pₖ/∂sⱼ=pₖ(1[k=j]−pⱼ)`이므로

```text
E[r] = Σ pₖrₖ
∂L/∂sⱼ = −pⱼ(rⱼ−E[r])
```

예시의 `E[r]=0.4×0.2+0.6×0.8=0.56`. 따라서 `∂L/∂s₁=+0.144`, `∂L/∂s₂=−0.144`다. 독립적인 score 좌표에서 gradient descent하면 첫 score는 내려가고 둘째는 올라간다. 평균보다 높은 reward 후보의 확률을 높이는 방향이다. 실제 모델은 파라미터를 공유하므로 모든 입력에서의 score 변화까지 독립적으로 보장되지는 않는다.

이는 원문 Eq. (2)에 대한 **해설자의 자체 유도**이며 논문에 같은 미분식이 적혔다는 뜻이 아니다. 고정된 후보에 대한 모델의 log probability를 통해 재작성기까지 gradient가 흐른다. 이 설명에는 이산 후보 생성·beam search·검색 순위에 대한 미분이나 고정 ANCE의 업데이트가 필요하지 않다.

배점: 유도 4점, 계산·부호·변화 방향 3점, gradient 경로와 가정 구분 3점. 다시 읽기: [원문 Eq. (2), Fig. 2](https://aclanthology.org/2024.naacl-long.449.pdf#page=4), [상세 설명](detailed-explanation.md).

</details>

## 11. 표를 과장하지 않고 읽기 — 8점

원문 Table 1의 **QReCC / sparse(BM25)** 일부다.

| 방법 | MRR | Recall@100 (%) |
|---|---:|---:|
| ConvGQR | 0.441 | 88.0 |
| IterCQR | 0.467 | 85.5 |

MRR의 절대 차이와 상대 변화율, Recall@100의 percentage point 차이를 구하라. “모든 검색 지표가 좋아졌다”와 “통계적으로 유의한 향상이다”라고 이 두 행만 보고 말할 수 있는가?

<details>
<summary>정답·이유·채점 기준</summary>

MRR 절대 차이는 `0.026`, 상대 변화율은 `0.026/0.441×100≈5.90%`. Recall@100은 `−2.5 percentage points`다. MRR과 recall이 서로 다르게 움직이므로 모든 지표 개선은 아니다. 두 행만으로 seed 분산·신뢰구간·검정 결과를 알 수 없으므로 통계적 유의성도 확정할 수 없다. 표의 값과 검증하고 싶은 결론을 구분해야 한다.

배점: MRR 절대·상대 차이 3점, recall 단위와 차이 2점, 과도한 결론 두 가지 경계 3점. 다시 읽기: [원문 Table 1, §4.1](https://aclanthology.org/2024.naacl-long.449.pdf#page=6), [논문 노트](README.md).

</details>

## 12. OOD 결과의 범위 — 5점

원문 Table 2에서 비교 모델은 ID, IterCQR는 OOD라고 표기한다. OOD가 여기서 뜻하는 학습·평가 설정을 설명하고, 이 표만으로 모든 도메인·한국어에 일반화한다고 주장할 수 없는 이유를 적어라.

<details>
<summary>정답·이유·채점 기준</summary>

IterCQR를 QReCC에서 학습해 TopiOCQA 테스트로 평가하거나 그 반대로 평가한다. 비교 모델은 해당 테스트 데이터셋의 in-domain 설정으로 보고됐다. 이는 두 영어 데이터셋 사이의 전이를 확인하는 근거이며, 모든 도메인·언어를 대표하지 않는다. ID와 OOD의 서로 다른 학습 조건도 표 해석에 남겨야 한다. 한국어 결과는 별도 실험 없이 만들 수 없다.

배점: 양방향 cross-dataset 설정 2점, 비교 조건 1점, 일반화 범위의 제한 2점. 다시 읽기: [원문 §4.2, Table 2](https://aclanthology.org/2024.naacl-long.449.pdf#page=6), [Ethical Statement](https://aclanthology.org/2024.naacl-long.449.pdf#page=9), [논문 노트](README.md).

</details>

## 13. Proxy reward 비판 — 5점

gold passage와의 cosine이 높아졌지만 실제 MRR은 낮아지는 가상 상황을 하나 구성하라. “보상이 잘못됐다”로 끝내지 말고 원인을 구분할 측정 하나를 제안하라. 저자가 명시한 한계와 자신의 비판을 구별해 적어라.

<details>
<summary>가능한 답·이유·채점 기준</summary>

예: rewrite가 넓은 주제 단어를 많이 포함하면서 gold와 cosine은 오르지만, 다른 문단들의 점수는 더 크게 올라 gold 순위가 내려갈 수 있다. BM25에서는 단어 일치가 달라져 dense 보상과 다른 결과가 날 수도 있다. 질문별 reward 변화와 reciprocal rank 변화를 함께 기록하고, 상위 경쟁 문단·rewrite 길이를 분석해 설명을 구분할 수 있다.

이는 가능한 실패를 설명한 **가상 반례와 해설 제안**이다. 논문이 이 상황을 그대로 관찰했다고 쓰면 안 된다. 저자가 명시한 한계는 dense reward가 sparse 검색을 일관되게 개선하지 않을 수 있다는 점 등이다. 실제 개인 비판은 학습자가 자신의 근거와 함께 작성한다.

배점: 성립 가능한 반례 2점, 구분 가능한 측정 2점, 저자·자신·가상 설명 구분 1점. 다시 읽기: [원문 Eq. (3)](https://aclanthology.org/2024.naacl-long.449.pdf#page=4), [Limitations](https://aclanthology.org/2024.naacl-long.449.pdf#page=9), [상세 설명](detailed-explanation.md).

</details>

## 14. 3개월 최소 연구 설계 — 7점

“dense proxy와 실제 검색 성능의 불일치”에서 출발해 연구 질문과 반증 가능한 가설을 작성하라. baseline 하나, 변경 하나, 데이터·split, 평가·비용, 필요한 자원, 월별 범위, 부정적 결과의 판단 기준을 포함하라. 관련 연구를 확인하기 전 신규성을 단정하지 말 것.

<details>
<summary>설계 예시·채점 기준</summary>

**정답이 하나인 문제가 아니다. 아래는 실행하지 않은 해설용 계획이며 학습자의 개인 연구 의견을 대신하지 않는다.**

예시 질문: “동일한 후보 집합에서 dense cosine만으로 Top-1을 고르는 대신 제한된 후보의 실제 BM25 순위를 이용하면 BM25 MRR이 개선되는가?” 가설은 “동일 예산에서 MRR이 개선되고, 추가 계산량을 포함한 비용 기준도 만족한다”처럼 측정 가능한 방향·판정 기준으로 좁힐 수 있다.

baseline은 공식 IterCQR 설정의 재현 가능한 부분으로 두고, 변경은 `t>τ`의 Top-1 **선택 기준 하나**로 한정한다. 후보 수·초기 모델·학습 예산·데이터를 고정한다. 학습 split의 gold만 후보 점수 계산에 사용하고, dev에서 설정을 정한 후 test는 최종 평가에 쓴다. MRR·Recall@100·질의별 변화·검색 횟수·학습 시간·메모리를 함께 기록한다. 작은 subset에서 개발했다면 그 범위를 명시한다.

1개월차: 관련 연구 확인·환경/데이터 확보·baseline 확인. 2개월차: 변경 하나와 동일 조건 비교. 3개월차: 반복·오류 분석·비용과 부정적 결과 정리. GPU·corpus index 저장공간·실행 시간을 먼저 확인하고, 부족하면 데이터·후보 평가 범위를 줄이되 비교 조건을 맞춘다. 개선이 없거나 비용 대비 이득이 없으면 가설을 지지하지 않는 결과로 보고한다. CONQRR 등 실제 검색 보상 선행 연구와의 차이는 원문을 읽고 검증해야 한다.

배점: 질문·반증 가능한 가설 1점, baseline·변경 하나 2점, 데이터·평가 누수 방지 1점, 지표·비용·자원 1점, 3개월 범위·부정적 결과 1점, 관련 연구·신규성 확인 1점. 다시 읽기: [원문 §2.1](https://aclanthology.org/2024.naacl-long.449.pdf#page=2), [§3.2.3](https://aclanthology.org/2024.naacl-long.449.pdf#page=5), [핵심 참고문헌](references.md), [연구 아이디어 양식](../../research-ideas.md).

</details>

## 답을 확인한 뒤 남길 기록

- 사전 / 사후 점수: [직접 작성] / [직접 작성]
- 아직 설명할 수 없는 단계와 원문 위치: [직접 작성]
- 정답을 보지 않고 새 예시로 설명해 본 내용: [직접 작성]
- 코드·검색 실행으로 확인할 가정: [직접 작성]

기초·흐름(1–7), 계산·gradient(8–10), 근거 해석·연구 판단(11–14)의 점수를 따로 살펴 부족한 부분부터 다시 읽는다. 높은 총점도 원문 대조·새 상황에의 적용·구현 검증을 대신하지 않는다.
