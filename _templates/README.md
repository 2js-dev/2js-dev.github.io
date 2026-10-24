# 블로그 작성 가이드

이 폴더(`_templates/`)는 언더스코어로 시작하므로 Jekyll 빌드에서 제외된다. 사이트에는 노출되지 않는다.

## 1. 폴더 구조

```
_posts/                     공개 글 (YYYY-MM-DD-slug.md)
_private/                   비공개 기록 (빌드 제외, 저장소에는 공개)
  projects/                 프로젝트 경험 정리
  inbox/                    트러블슈팅·공부 메모 수집함
_templates/                 글 템플릿 + 이 가이드
assets/img/posts/[slug]/    글별 이미지
_tabs/about.md              About 페이지
```

> `_private/`는 사이트에는 안 보이지만 저장소를 직접 보면 읽을 수 있다.
> 회사·고객사는 업종 + 알파벳(유통사 A, 결제사 B)으로 쓰고, 내부 시스템명·사내 코드는 일반화한다.

## 2. 카테고리

최대 2단계. 퍼머링크(`/posts/:title/`)에 카테고리가 들어가지 않으므로 나중에 바꿔도 링크가 깨지지 않는다.

| 대분류 | 소분류 | 용도 |
|---|---|---|
| Dev | Java, Spring, Database, Infra, CS | 개념·원리 정리, CS 공부 |
| Algorithm | Concept, Problem | 알고리즘 개념 / 문제 풀이 |
| Troubleshooting | Spring, Database, Infra, Build | 실제로 겪은 문제와 해결 |
| Retrospective | Project, Yearly | 공개 가능한 수준의 회고 |
| Daily | Travel, Life, Hobby | 일상 |

새 소분류는 기존 목록과 겹치지 않는지 확인한 뒤 이 표에 먼저 추가하고 사용한다.

## 3. 태그 규칙

- 영문 소문자, 단어 구분은 하이픈 (`spring-boot`, `github-actions`)
- 글 하나에 3~5개
- 같은 개념은 하나의 표기로 통일한다. 새 태그를 쓰기 전에 아래 사전과 사이트의 Tags 탭을 먼저 확인한다

| 분야 | 표준 태그 |
|---|---|
| 언어·프레임워크 | `java`, `jvm`, `spring`, `spring-boot`, `jpa`, `mybatis` |
| 데이터 | `mysql`, `oracle`, `redis`, `transaction`, `index` |
| 인프라 | `docker`, `linux`, `nginx`, `jenkins`, `github-actions`, `kafka` |
| CS | `os`, `network`, `data-structure` |
| 알고리즘 출처 | `baekjoon`, `programmers`, `leetcode` |
| 알고리즘 유형 | `bfs`, `dfs`, `dp`, `greedy`, `binary-search`, `two-pointer`, `sort`, `graph`, `implementation` |

혼동하기 쉬운 표기: `springboot`(X) → `spring-boot`, `ds`(X) → `data-structure`, `boj`(X) → `baekjoon`

## 4. 파일명·제목

- 파일명: `YYYY-MM-DD-slug.md`, slug는 영문 소문자 + 하이픈
- 트러블슈팅 제목: 증상이나 에러 키워드를 포함한다
  - X `Spring 트랜잭션 문제 해결`
  - O `@Transactional 내부 호출 시 롤백되지 않는 문제`
- 알고리즘 문제 제목: `[백준 1234] 문제명`, `[프로그래머스] 문제명`
- description: 검색 노출용 문구보다 "나중에 언제 이 글을 다시 보게 될지"를 기준으로 쓴다 (150자 이내)

## 5. 다시 찾기 위한 본문 규칙

- 환경과 버전을 명시한다 (Java, Spring Boot, DB, OS)
- 에러 로그는 원문 그대로 남긴다. 같은 에러를 만났을 때 메시지를 그대로 검색하면 찾을 수 있다
- 알고리즘 문제 지문은 복사하지 않고 링크와 한두 줄 요약만 남긴다

## 6. 작성 흐름

1. **메모**: 일이 생긴 당일, `_private/inbox/`에 에러 로그와 한 줄 메모만 남긴다
2. **초안**: 템플릿을 복사해 `_posts/`에 작성한다
3. **검토**: Claude에게 검토를 요청한다
4. **미리보기**: `bundle exec jekyll s`로 로컬 확인
5. **발행**: push → GitHub Actions 배포 확인

## 7. 템플릿

| 파일 | 용도 |
|---|---|
| `dev.md` | Dev 개념 글, Algorithm Concept 글 |
| `troubleshooting.md` | 트러블슈팅 |
| `algorithm-problem.md` | 알고리즘 문제 풀이 |

## 8. Chirpy 문법 메모 (7.6.0)

강조 박스는 인용문(`>`) 바로 아래에 클래스를 붙인다.

```markdown
> 핵심 팁 내용
{: .prompt-tip }
```

`prompt-tip`, `prompt-info`, `prompt-warning`, `prompt-danger`

코드 블록 파일명은 코드 블록 바로 아래에 `{: file="경로" }`를 붙인다.
