# ai-agent-playbook


# OpenCode 모델 라우팅 프리셋 공유

## 목적

OpenCode의 1.18.x의 경우 GPT 5.5 기반으로 모델설정이 되어있는데,  
GPT 5.6 기반으로 **작업 종류에 따라 잘 맞는 모델을 자동 선택**하도록 구성한 설정입니다.

예를 들면 일반 구현은 Sol, UI 작업은 Terra, 빠른 탐색은 Mini 모델을 쓰는 식입니다.

## 3줄 요약

1. Agent와 작업 유형별로 모델을 나눠두면 품질·속도·비용 균형을 맞추기 쉽습니다.  
2. 기본 모델이 잠시 실패하면 fallback 모델로 한 번만 자동 전환됩니다.  
3. 설정 적용 후 OpenCode를 재시작하고 새 세션에서 한 번 확인하면 충분합니다.

---

## 추천 모델 구성

### Agent별 기본 모델

| 용도 | 기본 모델 | 설명 |
|---|---|---|
| 일반 구현·계획·복잡한 작업 | `gpt-5.6-sol / high` | 기본 주력 모델 |
| 매우 어려운 추론 | `gpt-5.6-sol / xhigh` | 비용·시간은 더 들지만 깊은 추론용 |
| UI·디자인·시각 작업 | `gpt-5.6-terra / high` | 화면·UX 작업에 적합 |
| 빠른 코드 탐색 | `gpt-5.4-mini-fast` | 가볍고 빠른 검색·분석용 |
| 문서 작성 | `gpt-5.6-luna / medium` | 일반적인 글쓰기·정리용 |

### Category별 추천

| Category | 모델 |
|---|---|
| `visual-engineering` | Terra / high |
| `ultrabrain` | Sol / xhigh |
| `deep` | Terra / xhigh |
| `artistry` | Terra / high |
| `quick` | GPT-5.4-mini-fast |
| `unspecified-high` | Sol / high |
| `writing` | Luna / medium |

---

## 왜 GPT-5.6 계열을 쓰나요?

핵심은 “GPT-5.6이 GPT-5.5보다 무조건 비싸지만 더 좋다”가 아닙니다.

- **Sol**은 GPT-5.5와 API 토큰 단가가 동일한 수준이면서, 복잡한 추론·코딩 작업의 주력 모델로 사용할 수 있습니다.
- **Terra**는 Sol보다 약 50% 저렴해서 UI, 기획, 일반적인 고품질 작업을 맡기기 좋습니다.
- **Luna**는 반복 작업·문서 작성처럼 비용이 중요한 작업에 적합합니다.
- 따라서 하나의 모델만 고정하는 대신, 작업 난이도와 빈도에 맞춰 자동 분배하는 구성이 실용적입니다.

OpenAI의 모델 선택 가이드도 복잡한 추론·코딩에는 Sol, 품질과 비용의 균형에는 Terra, 대량·비용 민감 작업에는 Luna를 권장합니다.

---

## GPT-5.6 vs GPT-5.5 토큰 가격 비교

> 기준: OpenAI API Standard 요금, 짧은 컨텍스트, 100만 토큰당 USD  
> 가격은 변경될 수 있으므로 실제 운영 전에는 [공식 가격표](https://developers.openai.com/api/docs/pricing)를 다시 확인하세요.

| 모델 | 입력 100만 토큰 | Cached 입력 100만 토큰 | 출력 100만 토큰 | 추천 용도 |
|---|---:|---:|---:|---|
| GPT-5.6 Sol | $5.00 | $0.50 | $30.00 | 복잡한 구현·추론·주력 작업 |
| GPT-5.5 | $5.00 | $0.50 | $30.00 | 이전 세대 주력 모델 |
| GPT-5.6 Terra | $2.50 | $0.25 | $15.00 | UI·기획·일반 고품질 작업 |
| GPT-5.6 Luna | $1.00 | $0.10 | $6.00 | 문서·반복 작업·비용 민감 작업 |

### 한 번에 보는 비용 감각

입력 100만 토큰 + 출력 100만 토큰을 사용한다고 가정하면:

| 모델 | 대략적인 합계 |
|---|---:|
| GPT-5.6 Sol | $35.00 |
| GPT-5.5 | $35.00 |
| GPT-5.6 Terra | $17.50 |
| GPT-5.6 Luna | $7.00 |

즉, **Sol과 GPT-5.5는 표준 API 토큰 단가 기준으로 동일**합니다.  
그래서 Sol을 주력으로 쓰는 이유는 단순히 “더 비싼 최신 모델”이어서가 아니라, 같은 단가 범위에서 최신 모델 계열의 성능·토큰 효율을 활용하기 위해서입니다.

반대로 모든 작업을 Sol에 보내지 않고 Terra와 Luna를 함께 쓰는 이유는 분명합니다.

```text
복잡한 구현·추론      → Sol
일반 고품질 작업      → Terra
빠르고 반복적인 작업  → Luna 또는 Mini
```
이렇게 분배하면 품질이 필요한 작업에는 충분한 추론 예산을 쓰고, 단순 작업의 비용과 대기 시간은 줄일 수 있습니다.


## Fallback 모델이란?

Fallback은 기본 모델이 정상일 때는 전혀 사용되지 않는 **대체 경로**입니다.

예를 들어 Prometheus를 다음처럼 설정하면:

```text
기본: Sol / high
대체: Terra / high
```
평소에는 Sol/high로 실행됩니다.  
다만 모델 일시 오류, provider 장애, 제한 초과 등으로 실행할 수 없을 때만 Terra/high로 한 번 대체 시도합니다.
"runtime_fallback": {
  "enabled": true,
  "max_fallback_attempts": 1
}
즉, 무한 재시도나 여러 모델 동시 실행이 아니라 작업이 바로 멈추지 않도록 하는 안전장치입니다.

##적용 후 한 번만 해둘 것

OpenCode TUI에서 이전에 직접 고른 모델 variant가 남아 있으면 agent 설정을 덮어쓸 수 있습니다.

설정을 적용한 뒤 아래 순서만 한 번 해주세요.

/models    → gpt-5.6-terra 선택
/variants  → high 선택

/models    → gpt-5.6-sol 선택
/variants  → default 선택

/exit
그 다음 터미널에서 다시 시작합니다.
opencode
새 대화가 필요하면:
/new

###다른 OpenCode에 넣을 프롬프트


아래 프롬프트를 프로메테우스에 그대로 붙여 넣으면, 과한 감사 절차 없이 적용 계획을 만들도록 요청할 수 있습니다.

```
OpenCode와 oh-my-openagent에 모델 routing 설정을 적용할 짧고 안전한 작업 계획을 작성해주세요.

목표:
- 기존 설정을 보존하면서 agent/category별 모델 routing만 병합
- 적용 후 OpenCode 재시작 및 새 세션 1회 확인
- 과도한 보안 감사, 다중 reviewer, 긴 session DB 분석은 제외
- 지금은 실행하지 말고 적용 계획만 작성

기본 원칙:
- 기존 config 파일을 통째로 덮어쓰지 말 것
- 변경 전 대상 config 파일만 backup 생성
- JSON 유효성 확인 후 적용
- 모르는 모델 이름이나 누락된 plugin은 임의로 대체하지 말고 보고
- secret, API key, transcript, session DB는 건드리지 말 것
- 실행 중인 OpenCode 프로세스를 강제 종료하지 말 것

사용할 모델:
- Sol: openai/gpt-5.6-sol
- Terra: openai/gpt-5.6-terra
- Luna: openai/gpt-5.6-luna
- Mini Fast: openai/gpt-5.4-mini-fast
- Mini: openai/gpt-5.4-mini

Agent routing:
- sisyphus, hephaestus, prometheus: Sol/high → Terra/high
- oracle: Sol/xhigh → Terra/xhigh
- librarian, explore: Mini Fast → Mini
- multimodal-looker: Luna/medium → Terra/medium
- metis, momus, atlas, sisyphus-junior: Terra/high → Sol/high

Category routing:
- visual-engineering: Terra/high → Sol/high
- ultrabrain: Sol/xhigh → Terra/xhigh
- deep: Terra/xhigh → Sol/high
- artistry: Terra/high → Sol/high
- quick: Mini Fast → Mini
- unspecified-low: Luna/medium → Terra/medium
- unspecified-high: Sol/high → Terra/high
- writing: Luna/medium → Terra/medium

Runtime fallback:
- enabled: true
- max_fallback_attempts: 1

검증은 가볍게 진행해주세요:
1. JSON 파싱 확인
2. `opencode debug config`에서 routing 확인
3. OpenCode 재시작
4. 새 Prometheus 세션 한 번만 생성해서 Sol/high 확인

계획은 3~4단계로 짧게 작성하고,
각 단계에 변경 대상·확인 방법·실패 시 복구 방법만 포함해주세요.
```

###한 줄 결론
이 설정은 “무조건 가장 강한 모델 하나”를 쓰거나 오픈코드 1.18.x 버전에서 세팅된 초기 GPT 5.5를 포함하는 에이전트 운용 방식이 아니라,  
GPT 5.6 기반으로 작업에 맞는 모델을 맞춰 쓰고 문제가 있을 때만 비슷한 급의 모델로 한 번 대체하는 실용적인 OpenCode 프리셋입니다.
