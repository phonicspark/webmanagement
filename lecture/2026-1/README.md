# AI 리터러시 스쿨 1회차

## 카파시의 LLM Wiki로 나만의 AI 세컨드 브레인 만들기

HTML 강의 페이지: [GitHub Pages에서 보기](https://phonicspark.github.io/webmanagement/2026-1/01_second_brain.html)

자료 위치: [WEBMANAGEMENT 2026-1](https://github.com/phonicspark/webmanagement/tree/master/2026-1)

참고 원문: [Karpathy LLM Wiki Idea File](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)

### 핵심 메시지

RAG는 매번 검색해서 답을 만듭니다. LLM Wiki는 지식이 쌓이기 때문에 시간이 지날수록 답이 더 좋아집니다.

개인 지식관리에는 단순 검색보다 축적이 필요합니다. 중요한 것은 자료를 많이 저장하는 것이 아니라, 내 목적에 맞게 모으고, AI가 읽을 수 있는 위키로 컴파일하고, 다시 질문과 결과물로 연결하는 것입니다.

### 오늘 만들 것

이번 회차에서는 Karpathy의 LLM Wiki 패턴을 바탕으로 Claude Code, Obsidian, Obsidian Web Clipper, Graphify를 연결한 개인 AI 세컨드 브레인 흐름을 설계합니다.

수업이 끝나면 다음을 이해하고 직접 시작할 수 있습니다.

- RAG와 LLM Wiki의 차이
- `raw/`, `wiki/`, `Output/` 구조
- AI가 매번 읽는 설명서 `CLAUDE.md`
- 웹 클리퍼로 수집하는 방법
- 인제스트로 자료를 내 지식으로 바꾸는 방법
- `/ingest`, `/query`, `/lint` 같은 반복 작업 스킬화
- Graphify로 위키를 지식 그래프로 바꾸는 방법

### 사전 준비

필요한 도구는 다음과 같습니다.

- Obsidian: 새 볼트 생성
- Claude Code: 데스크탑 앱 또는 터미널에서 볼트 폴더에 연결
- Obsidian Web Clipper: 크롬 확장 프로그램
- Graphify: Python 3 환경에서 `pip install graphifyy`

시작 전 숙제는 한 줄씩이면 충분합니다.

1. 나는 누구인가: 이름, 하는 일, 역할
2. 왜 기록하고 싶은가: 지금 무엇이 안 되는지, 기록이 되면 무엇이 달라지는지
3. 어떤 아웃풋을 만들고 싶은가: 누구를 위해, 어떤 형태로 만들고 싶은지

이 세 가지가 없으면 AI는 목적 없이 정리만 합니다. 목적 있는 수집만이 좋은 데이터가 됩니다.

### LLM Wiki 핵심 아이디어

일반적인 RAG는 파일을 올려두고 질문할 때마다 관련 조각을 찾아 답을 만듭니다. 이 방식은 편하지만 질문할 때마다 지식을 다시 조립해야 합니다. 미묘한 질문, 여러 문서가 엮인 질문, 시간이 지나며 누적되는 관점에는 약합니다.

LLM Wiki는 다릅니다. 원본 자료는 `raw/`에 보관하고, AI가 그 자료를 읽어 `wiki/` 안의 마크다운 문서로 계속 정리하고 연결합니다. 새 자료가 들어오면 AI는 단순히 색인만 하지 않고 기존 개념 페이지, 엔티티 페이지, 요약 페이지, 인덱스, 로그를 업데이트합니다.

비유하면 Obsidian은 IDE, LLM은 프로그래머, wiki는 코드베이스입니다. 사람은 좋은 소스를 고르고 방향을 잡습니다. AI는 요약, 연결, 정리, 로그 기록 같은 유지보수를 맡습니다.

### Step 1. 맥락 인터뷰

먼저 AI가 나의 목적을 이해해야 합니다. 숙제 3가지를 바탕으로 AI에게 인터뷰를 요청합니다. 음성 입력으로 편하게 답해도 됩니다.

```text
우리는 지금 "AI를 위한 세컨드 브레인"을 만들고 있어.

목적: 세상에 있는 유용한 정보와 지식을 체계적으로 모으고,
나의 맥락에 맞게 정리·연결해서, AI가 이를 읽고 활용할 수 있게 만드는 것.

'나의 핵심 맥락.md'는 내가 사전에 작성한 자기소개야.
이걸 읽고, 나의 맥락을 더 깊이 이해하기 위해 인터뷰해줘.

아래 순서로 한 번에 하나씩 질문해줘:
1. "나는 누구인가"를 더 깊이: 역할, 강점, 가치관
2. "왜 기록하고 싶은가"를 더 깊이: 지금 안 되는 것, 비전
3. "어떤 아웃풋을 만들고 싶은가"를 더 깊이: 대상, 형태, 1년 후 이상적인 모습

각 질문에 대한 내 답변을 받으면:
- 깔끔하게 요약해줘
- 다음 질문으로 넘어가줘

3개 질문이 끝나면:
→ 전체 답변을 종합해서 "나의 맥락 요약"을 만들어줘.
```

### Step 2. CLAUDE.md 생성

인터뷰 결과가 `나의 핵심 맥락.md`에 정리되면, 이를 바탕으로 `CLAUDE.md`를 만듭니다. 이 파일은 AI가 매번 읽는 나의 설명서입니다.

```text
내 볼트에 "나의 핵심 맥락.md" 파일이 있어.
이 파일을 읽고, CLAUDE.md를 만들어줘.

포함할 섹션:
1. "## 나는 누구인가" → 이름, 하는 일, 핵심 가치
2. "## 나의 역할들" → 각 역할별: 하는 일, 주요 관심사
3. "## 나의 비전과 목표" → 이루고자 하는 것, 타겟 독자/고객
4. "## AI에게 기대하는 것" → 어떤 도움을 원하는지
5. "## 작업 규칙" → 톤, 언어, 결과물 형태

스타일:
- 간결하게, 핵심만
- AI가 매번 읽는다고 생각하고 작성
- 빈칸이 있어도 괜찮아. 채워진 것만으로 만들어줘.
```

대화가 끝나도 AI가 나를 더 잘 이어서 도울 수 있는 이유가 이 파일에 있습니다.

### Step 3. LLM Wiki 세팅

Karpathy의 LLM Wiki Idea File을 AI에게 전달하고, 내 `CLAUDE.md`에 맞춘 구조를 만들도록 요청합니다.

```text
아래는 Karpathy의 "LLM Wiki" 패턴이야.
내 CLAUDE.md를 읽고, 이 아이디어를 나에게 맞춰서 세팅해줘.

[Karpathy Idea File 전문을 붙여넣기]
원본: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

해줄 것:
1. 내 목적에 맞는 폴더 구조 만들기
   - raw/ → 불변 원본, 내 인풋 유형별 하위 폴더
   - wiki/ → AI가 컴파일하는 위키
   - Output/ → 결과물
2. CLAUDE.md에 위키 운영 규칙 추가
3. 각 주요 폴더에 하위 CLAUDE.md 생성
4. wiki/index.md 초기화
5. wiki/log.md 초기화
```

생성되는 기본 구조는 다음과 같습니다.

```text
볼트/
├── CLAUDE.md
├── raw/
│   ├── articles/
│   ├── videos/
│   ├── podcasts/
│   ├── books/
│   └── CLAUDE.md
├── wiki/
│   ├── index.md
│   ├── log.md
│   └── CLAUDE.md
└── Output/
    └── CLAUDE.md
```

운영 규칙 10가지는 반드시 기억합니다.

- `raw/`는 불변 원본이며 절대 수정하지 않습니다.
- wiki 페이지를 만들거나 지우면 `wiki/index.md`를 업데이트합니다.
- 모든 작업은 `wiki/log.md`에 기록합니다.
- 내부 참조는 Obsidian wikilink 형식으로 씁니다.
- 모든 wiki 페이지에는 YAML frontmatter를 둡니다.
- 모순을 발견하면 양쪽 소스를 모두 인용합니다.
- 소스 요약은 사실만 담고, 해석은 개념 페이지에서 다룹니다.
- 질문에 답할 때는 `wiki/index.md`를 먼저 보고, `raw/`는 마지막 수단으로 봅니다.
- 새 페이지를 만들기보다 기존 페이지 업데이트를 우선합니다.
- 인덱스 항목은 한 줄로, 120자 이내로 유지합니다.

### Step 4. 웹 클리퍼 템플릿

Obsidian Web Clipper의 기본 템플릿 대신 LLM Wiki에 맞는 템플릿을 만듭니다.

진행 순서는 다음과 같습니다.

1. Chrome 웹스토어에서 Obsidian Web Clipper를 설치합니다.
2. 확장 프로그램을 고정합니다.
3. 웹 클리퍼 설정에서 기본 템플릿을 JSON으로 내보냅니다.
4. 내보낸 JSON을 Claude Code에 전달합니다.
5. Article, YouTube, Podcast, Book, Research용 템플릿을 생성합니다.
6. 생성된 JSON을 웹 클리퍼에 다시 들여옵니다.

```text
지금 보고 있는 파일은 옵시디언 웹 클리퍼 기본 템플릿인데,
내 raw/ 폴더에 맞는 템플릿을 만들어줘.

이 JSON 파일을 여러 가지로 만들어줬으면 좋겠어:
- 아티클 (Article)
- 유튜브 (YouTube)
- 팟캐스트 (Podcast)
- 책 (Book)
- 연구 자료 (Research)

이 템플릿들로 수집을 했을 때,
LLM Wiki로 인제스트를 요청했을 때
wiki로 잘 소화할 수 있도록 우리가 정한 스키마에 맞게 해줘.
```

유튜브 같은 특정 사이트는 자동 트리거를 설정할 수 있습니다. 목표는 버튼 한 번으로 원본이 `raw/videos/` 같은 위치에 저장되게 만드는 것입니다.

### Step 5. 인제스트

모으기만 하면 지식이 아니라 저장소입니다. 내 생각과 목적이 붙어야 지식이 됩니다.

대화형 인제스트 프롬프트:

```text
방금 웹 클리퍼로 raw/에 저장한 글이 있어. 인제스트해줘.

1. 해당 파일을 읽어줘. raw/는 절대 수정하지 마.
2. 소스 요약을 만들어줘:
   - 핵심 주장 (Key Claims)
   - 언급된 엔티티 (사람, 도구, 조직)
   - 다루는 개념 (Concepts)
3. 나에게 질문해줘:
   - 이 글을 왜 캡처했어?
   - 지금 하고 있는 일과 어떻게 연결돼?
   - 이걸로 뭘 해보고 싶어?
4. 내 답변을 받으면 wiki/에 반영해줘:
   - 소스 요약 페이지 생성
   - 관련 엔티티/개념 페이지 업데이트
   - index.md, log.md 업데이트
```

빠른 인제스트 프롬프트:

```text
raw/ 폴더를 스캔해서 아직 인제스트 안 된 파일을 모두 처리해줘.
대화 없이 바로 wiki/에 반영하고, 끝나면 요약을 보여줘.
```

가장 중요한 질문은 "내가 이걸 왜 수집했는가?"입니다. 이 질문에 답할 수 없으면 목적 없는 수집입니다.

### Step 6. 스킬 만들기

2~3번 수동으로 해보고 패턴이 잡히면 반복 작업을 스킬로 만듭니다.

```text
지금까지 대화한 내용을 바탕으로 스킬을 만들어줘.

새로운 최신 파일들을 확인하고 인제스트해주는
지금까지 한 모든 프로세스를 포함하는 /ingest 스킬을 만들어줘.
```

추가로 만들면 좋은 스킬은 다음과 같습니다.

- `/query`: 질문을 받으면 `wiki/` 문서를 기반으로 답변합니다. 벡터 DB가 아니라 위키 문서를 읽고 답합니다.
- `/lint`: 위키 전체에서 깨진 링크, 오래된 주장, 고립된 문서, 누락된 연결을 찾아 정리합니다.

스킬을 너무 빨리 만들기보다 수동으로 몇 번 해본 뒤 만드는 것이 좋습니다. 그래야 AI가 내 실제 패턴을 반영할 수 있습니다.

### Step 7. Graphify

`wiki/` 문서가 많아지면 키워드 검색만으로는 관계를 보기 어렵습니다. Graphify는 위키 문서를 그래프 구조로 바꿔 노드와 엣지 기반으로 지식을 탐색하게 도와줍니다.

설치:

```bash
pip install graphifyy
```

실행:

```text
/graphify wiki/
```

생성되는 결과물:

| 파일 | 용도 |
| --- | --- |
| `graphify-out/graph.json` | AI가 참조하는 그래프 데이터 |
| `graphify-out/graph.html` | 브라우저에서 보는 시각화 |
| `graphify-out/GRAPH_REPORT.md` | 그래프 분석 리포트 |

그래프 기반 질의:

```text
/graphify query "브레인 트리니티와 LLM Wiki의 관계는?"
```

증분 업데이트:

```text
/graphify wiki/ --update
```

일반 Query가 텍스트 키워드로 관련 문서를 찾아 답한다면, Graphify Query는 노드와 엣지의 관계를 탐색해 의미적 연결을 바탕으로 답합니다.

### 전체 흐름 요약

```text
[숙제 3가지 답변]
    ↓
[Step 1] 맥락 인터뷰 → "나의 핵심 맥락.md"
    ↓
[Step 2] CLAUDE.md 생성 → AI가 나를 아는 설명서
    ↓
[Step 3] LLM Wiki 세팅 → raw/ + wiki/ + Output/ 폴더 자동 생성
    ↓
[Step 4] 웹 클리퍼 템플릿 → 수집 자동화
    ↓
[Step 5] 인제스트 → 수집한 것을 wiki/에 소화
    ↓
[Step 6] 스킬 만들기 → /ingest, /query, /lint 자동화
    ↓
[Step 7] Graphify → 지식 그래프로 관계 기반 활용
```

### 오늘의 결과물

- `나의 핵심 맥락.md` 초안
- AI가 읽을 `CLAUDE.md` 설계안
- `raw/`, `wiki/`, `Output/` 폴더 구조
- `wiki/index.md`, `wiki/log.md` 초기 형태
- 웹 클리퍼 템플릿 생성 프롬프트
- 인제스트 프롬프트 2종
- Graphify 실행 흐름

### 제일 중요한 것

LLM이 알아서 해주는 것이 아닙니다. 나의 목적이 있어야 합니다.

목적 없이 수집하면 쓰레기 데이터가 됩니다. 목적 있게 수집하면 골드 데이터가 됩니다.

Gold In, Gold Out.

### 참고 링크

- [Karpathy LLM Wiki Idea File](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [Graphify 소개](https://aicoolies.com/tools/graphify)
- [Obsidian Web Clipper](https://chromewebstore.google.com/search/Obsidian%20Web%20Clipper)
