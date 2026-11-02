# experiment

2026-08-25에 수행한 다섯 `shelf_fit` 프롬프트 비교를 재현·보관하는 영역입니다.
실제 데모 구현과 프로젝트 문서의 정본은 `../real/`에 있습니다.

- `evaluate.py`: 표본 도서(골든 시나리오) 반복 평가
- `app_for_experiment.py`: 프롬프트를 화면에서 바꿔 보는 작업창
- `data/scenarios/`: 입력·작성자안·교열안·명시적 holdout ID
- `prompts/`: shelf_fit 비교안 5종
- `scripts/`: 고정 후보 비교 실행기와 결과 분석기
- `output/`: 원물 JSON과 사람이 읽는 요약

운영 코드는 `../real/src/`를 그대로 import합니다. 실험용 로직을 복사해 두 벌로 만들지 않습니다.

## 현재 결론

작성자와 교열이 일치한 표본 도서를 3회씩 평가했을 때 `two_axis`가 교열 최종 서가를 약 92% 1위로 두어
다섯 버전 중 가장 좋았습니다. 이 파일은 `../real/prompts/systemprompt_shelf_fit.txt`에
최종 운영안으로 반영되어 있습니다. 비교에 쓴 원본을 보존하기 위해
`prompts/exp_shelf_fit_two_axis.txt`도 그대로 둡니다.

2026-09-02에 이 회차의 원물(`output/…_comparison.json`)을 다시 읽어 운영 임계값을
정했습니다 — `THRESHOLD_2=0.80` · `GAP_MIN=0.15`(둘 다 넘어야 수렴). 근거와 케이스별
판정은 [`../docs/design.md`](../docs/design.md) §3과 [`../real/README.md`](../real/README.md)에 있습니다.
⚠️ 같은 표본에 되적용한 in-sample 값이라 **잠정**이며, 기준선 회차에서 재확인합니다.
상세 결과는 `output/20260825-140743_shelf_prompt_comparison_summary.md`를 봅니다.

**평가 실행기는 2단계 게이트를 끕니다**(`decide.STAGE2_GATE = False`) — 게이트가 닫히면
그 케이스의 키워드 후보에 점수가 아예 안 생겨 분포에 구멍이 나기 때문입니다. 운영
(`real/app.py`·`run.py`)은 기본 on입니다. 회차 조건에 `stage2_gate`로 기록됩니다.

## 재현

```bash
python evaluate.py --label=myrun --runs=3
streamlit run app_for_experiment.py
python scripts/compare_shelf_fit_prompts.py --runs 3
python scripts/analyze_shelf_fit_comparison.py output/<comparison.json>
```

## 평가 결과 읽는 법

`python evaluate.py`를 돌리면 `output/`에 회차별 결과가
쌓입니다(덮어쓰지 않습니다).

```
output/<회차>_summary.md     사람이 읽는 비교 요약
output/<회차>.json           케이스별 상세·프롬프트 trace
```

**채점은 책의 종류에 따라 두 갈래다:**

| 유형 | 무엇으로 채점하나 |
|---|---|
| **정답이 하나였던 책** (작성자 == 교열) | 1순위가 그 번호인가 |
| **사람도 갈렸던 책** (작성자 != 교열) | ① 경합한 두 번호를 후보에 다 담았나 ② 사서 확인이 필요하다고 표시했나 |

후자는 *맞히는* 게 아니라 **판단을 사람에게 제대로 넘기는 것**이 성공이다.

> ⚠️ **LLM 판단이라 실행할 때마다 결과가 흔들린다.** 단일 회차 숫자로 판단하지 말고, 같은 조건으로 2~3회 돌려 어떤 케이스가 고정이고 어떤 게 흔들리는지 보는 것이 맞다. 실제로 2026-08-05에 같은 조건 두 회차가 10/12와 9/12로 갈렸고 「부린 왕자」는 정답↔오답이 뒤집혔다.
