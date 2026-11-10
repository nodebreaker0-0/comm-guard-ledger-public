# comm-guard (공개판)

메시지와 문서를 보내기 전에 점검하는 Claude 스킬입니다. 초안을 사고 완결(C0)과 여섯 축(결론 선행, 길이, 탑다운 구조, 독자, 대응 모드, 요청 액션)으로 판정하고, 판정 결과와 상대의 실제 반응을 한 줄씩 기록해 다음 점검의 자료로 씁니다.

## 이 저장소는 공개용으로 따로 만든 것입니다

실제로 쓰는 원본은 비공개 저장소(`comm-guard-ledger`)에 있습니다. 원본에는 판정 기록과 상대 모델 같은 상태 파일이 있고, 그 안에 동료의 이름과 사내 대화 등 개인정보가 들어 있어 공개할 수 없습니다. 그래서 스킬 문서와 안내 문서만, 개인정보와 사내 정보를 지워 이 저장소로 옮겼습니다.

| 파일 | 비공개 원본 | 이 저장소 |
|---|---|---|
| `SKILL.md` | 실제 사용본 | 공개판. 사람 이름, 특정 인물에 대한 모델, 사내 스레드·회의·문서 이름을 지우거나 일반화 |
| `README.md` | 원본 안내 | 이 문서 |
| `ledger.jsonl` | 판정 기록 (판정 1건 = 1줄) | 싣지 않음 |
| `state/*.md` | 상대 모델, 인물평가성 지적 대응 플레이북, 자기 모델 | 싣지 않음 |

`SKILL.md` 본문의 "사용자"는 원본에서 작성자 본인을 가리킵니다.

## 스킬 구성

| 층 | 내용 |
|---|---|
| C0 | 사고 완결 게이트. 모든 발신에 자가체크 4문항, 선제 발신에는 심문(probe) 6문항 |
| C1~C6 | 결론 선행 / 길이·밀도 상한 / 탑다운 구조 / 독자 렌즈 / 대응 모드 / 요청 액션 명확성 |
| P1~P6 | 상황별 프로토콜: 지적 대응, 방향 모호, 스코프 변경, 중간보고, 질문 대응, 인물평가성 지적 대응 |
| R-lens | 반복해서 소통하는 상대의 모델. 예측을 등록하고 실제 반응과 대조해 고침 |
| ledger | 모든 판정을 기록하고 주기적으로 리뷰 |

## 기록 형식 (ledger 스키마)

원본 저장소의 `ledger.jsonl`은 판정 1건을 1줄로 적고, 행을 지우지 않습니다. 실제 반응이 도착하면 해당 행의 `feedback`만 고칩니다.

| key | 타입/값 |
|---|---|
| `date` | ISO date |
| `subject` | 발신물 제목 |
| `phase` | `사전` \| `사후` \| `리뷰` |
| `purpose` | 판단요청/컨펌/FYI/질문답변/지적대응(P1)/싱크요청(P2)/스코프싱크(P3)/공유멘트 |
| `audience` | 리더십/팀/외부/기타 |
| `verdict` | `PASS` \| `REWRITE` \| `HOLD(P2)` \| null(리뷰행) |
| `tags` | ⊂ {#중요도 #카테고리 #흐름추상화 #큰그림 #탑다운 #방어 #독자망각} |
| `proactive` | bool (선제발신) |
| `probe_gap` / `feedback` / `link` / `memo` | text \| null |

커밋 메시지 규칙: `ledger: {date} {verdict} {subject 30자}` / `feedback: {subject 30자}` / `state: {file} {v}`

## 리뷰 레시피

```bash
jq -r '.tags[]?' ledger.jsonl | sort | uniq -c | sort -rn                        # 태그 쏠림
jq -r 'select(.verdict=="REWRITE") | .date' ledger.jsonl | sort | uniq -c        # REWRITE 추이
jq -r '[.proactive, .verdict] | @tsv' ledger.jsonl | sort | uniq -c              # 선제 vs 반응
jq 'select(.probe_gap != null and .feedback != null) | {subject, probe_gap, feedback}' ledger.jsonl  # probe 예측률
jq -r '[.purpose, .verdict] | @tsv' ledger.jsonl | sort | uniq -c | sort -rn     # 용도×판정
grep -c '예측' ledger.jsonl                                                       # R-lens 예측 등록 수
```

## 기록 요약 (비공개 원본, 2026-07-31 기준)

- 기간과 분량: 2026-07-14 ~ 07-31, 41행 (초안·발신 판정 33행 + 리뷰 8행)
- 초안·발신 33행의 판정: PASS 19 · REWRITE 13 · HOLD 1
- 다시 쓴 13건의 원인 태그 18개 가운데 15개가 나열 계열 (#카테고리 5 · #탑다운 4 · #흐름추상화 3 · #중요도 2 · #큰그림 1)
- 스킬 버전: v1 (07-17) → v1.1 (07-18) → v2.0 (07-23)
