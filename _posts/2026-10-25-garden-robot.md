---
title: "[구현·시뮬레이션] 정원 관리 로봇"
date: 2026-10-25 14:00:00 +0900
categories: [Algorithm, Problem]
tags: [implementation, java]
description: "'동시에 갱신'과 '방향 회전'을 다루는 격자 시뮬레이션 문제"
---

풀이는 **①슈도코드 → ②디버깅 → ③최종 코드** 세 단계로 진행한다.

***

## 📌 문제

| 항목 | 내용             |
| ---- | ---------------- |
| 출처 | Claude 출제      |
| 분류 | 구현, 시뮬레이션 |
| 결과 | 성공             |

`N×M` 정원의 각 칸에 높이 0~9의 잡초가 있고(0은 빈 땅), 로봇이 `(r, c)`에서 방향 `d`(0 북, 1 동, 2 남, 3 서)를 보고 서 있다. 1초마다 아래 세 단계가 **순서대로** 일어난다.

1. **성장**: 높이 1 이상인 잡초는 모두 +1 (최대 9)
2. **번식**: 성장이 끝난 상태 기준으로, 높이 0인 칸 중 상하좌우에 높이 5 이상 잡초가 2개 이상이면 높이 1이 된다. 모든 칸에서 **동시에** 판정한다.
3. **로봇 행동**
   - 앞 칸이 정원 밖이면 이동하지 않고 시계 방향으로 90도 회전
   - 앞 칸이 정원 안이면 이동해서 잡초를 깎아 0으로 만든다. 깎은 높이가 7 이상이면 반시계 방향으로 90도 회전

`K`초 뒤 **정원 전체 잡초 높이의 합**과 **로봇이 깎은 높이의 총합**을 출력한다.
- 로봇이 처음 서 있는 칸은 깎지 않은 상태로 시작한다

##### 입력
```bash
첫 줄: N M K   (3 ≤ N, M ≤ 50, 1 ≤ K ≤ 1,000)
둘째 줄: r c d (로봇의 시작 위치와 방향, 0-indexed)
다음 N줄: 각 칸의 잡초 높이 M개 (0 ~ 9)
```
{: file="input" }

##### 입력 예시
```bash
4 5 6
1 1 1
0 3 0 0 0
2 0 6 4 0
0 6 0 0 1
0 0 0 0 0
```
{: file="input example" }

##### 출력 예시
```bash
70 7
```
{: file="output example" }

***

## 🔍 접근

- 칸 수(`N×M`)는 최대 2,500, 시간(`K`)은 최대 1,000초다. 매초 격자 전체를 몇 번 훑어도 수천만 번 안쪽이라 **규칙을 그대로 시뮬레이션**하면 된다.

***

## 📝 풀이 과정

### 1단계: 슈도코드

**1초 동안 일어나는 일**을 문제 순서 그대로 구현해본다. 반복문(K초)을 제외하고 한 번의 사이클만 일어난다고 가정해서 작성했다.

```java
// [ACTION per second]
// 1. 높이가 1 이상인 모든 잡초 높이 1 증가 (단, 최대 높이는 9)
for(int i = 0; i < N; i++) {
    for(int j = 0; j < M; j++) {
        garden[i][j] = garden[i][j] >= 1 && garden[i][j] < 9 ? garden[i][j] + 1 : garden[i][j];
    }
}

// 2. 높이가 0인 칸은 상하좌우에 높이 5 이상 잡초가 2개 이상이면 해당 칸은 높이 1로 증가(로봇 방향과 상관 없음)
for(int i = 0; i < N; i++) {
    for(int j = 0; j < M; j++) {
        if(isZeroHeight[i][j]) {
            int count = 0;
            if(i + dr[0] >= 0 && garden[i + dr[0]][j] >= 5) {
                count++;
            }
            if(i + dr[2] < N && garden[i + dr[2]][j] >= 5) {
                count++;
            }
            if(j + dc[1] < M && garden[i][j + dc[1]] >= 5) {
                count++;
            }
            if(j + dc[3] >= 0 && garden[i][j + dc[3]] >= 5) {
                count++;
            }

            if(count >= 2) {
                garden[i][j] = 1;
            }
        }
    }
}

// 3. 로봇 행동
// - 바라보는 방향의 앞 칸이 정원 밖이면 이동하지 않고 시계 방향으로 90도 회전
// - 앞 칸이 정원 안이면 그 칸으로 이동하고, 그 칸의 잡초를 깎아 높이를 0으로 변경 + 깎은 높이가 7 이상이면 반시계 방향 90도 회전
// - 방향: 0 = 북(위), 1 = 동(오른쪽), 2 = 남(아래), 3 = 서(왼쪽)
boolean isOutside = false;
switch(d) {
    case 0 -> isOutside = r + dr[d] < 0;
    case 2 -> isOutside = r + dr[d] >= N;
    case 1 -> isOutside = c + dc[d] >= M;
    case 3 -> isOutside = c + dc[d] < 0;
}

if(isOutside) {
    d += 1;
} else {
    switch(d) {
        case 0 -> r = r + dr[d];
        case 2 -> r = r + dr[d];
        case 1 -> c = c + dc[d];
        case 3 -> c = c + dc[d];
    }
    if(garden[r][c] >= 7) {
        d -= 1;
    }
    garden[r][c] = 0;
}
```
{: file="PseudoCode" }

- 성장 단계에서 0인 칸이 그대로 0으로 남으므로 원래 빈 땅이었던 칸을 따로 기억하려고 `isZeroHeight` boolean 배열을 두었다.
- 방향 배열 `dr`, `dc`를 북·동·남·서 순서로 맞춰 두면 `d`를 그대로 인덱스로 쓸 수 있다.

⚠️ 이 단계에서 놓친 것들이 있었는데, 다음 단계에서 로그를 찍으며 드러났다.

### 2단계: 디버깅

슈도코드를 완성코드(K초)로 변경 후 **각 단계가 끝날 때마다 격자를 출력**하도록 했다.

출력 로그을 보면서 고친 부분은 다음과 같다.

```diff
# 2번
if(isZeroHeight[i][j]) {
    ...
    if(count >= 2) {
        garden[i][j] = 1;
+       isZeroHeight[i][j] = false; // 잡초가 자라면 빈 땅이 아니어야 함
    }
}

# 3번
if(isOutside) {
-    d += 1;
+    d = d + 1 > 3 ? 0 : d + 1; // 시계 방향 서 → 북 이동 시 3 → 0 으로 바뀌어야 함
 } else {
     ...
     if(garden[r][c] >= 7) {
-        d -= 1;
+        d = d - 1 < 0 ? 3 : d - 1; // 반시계 방향 북 → 서 이동 시 0 → 3 으로 바뀌어야 함
     }
     garden[r][c] = 0;
+    isZeroHeight[r][c] = true; // 빈 땅 표시
 }
```

1. **방향 범위 초과**: 서(3)에서 시계 회전하면 `d = 4`, 북(0)에서 반시계 회전하면 `d = -1`이 되어 `dr[d]`에서 `ArrayIndexOutOfBoundsException`이 난다. 0~3을 순환하도록 고쳤다.
2. **`isZeroHeight` 동기화 누락**: 번식으로 1이 된 칸은 `false`로, 로봇이 깎은 칸은 `true`로 갱신해야 한다. 슈도코드에서는 이 갱신이 빠져 있어서 깎인 칸이 다시 번식하지 못했다.

### 3단계: 최종 제출 코드

디버깅 코드에서 로그만 걷어낸 버전이다. 

```java
private static void solution() throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
    StringTokenizer st;
    StringBuilder result = new StringBuilder();
    int[][] garden;
    boolean[][] isZeroHeight;
    int[] dr = {-1, 0, 1, 0}; // 북, 동, 남, 서
    int[] dc = {0, 1, 0, -1}; // 북, 동, 남, 서

    // [INPUT]
    int N = 0, M = 0, K = 0;
    int r = 0, c = 0, d = 0;

    st = new StringTokenizer(br.readLine());
    N = Integer.parseInt(st.nextToken());
    M = Integer.parseInt(st.nextToken());
    K = Integer.parseInt(st.nextToken());

    st = new StringTokenizer(br.readLine());
    r = Integer.parseInt(st.nextToken());
    c = Integer.parseInt(st.nextToken());
    d = Integer.parseInt(st.nextToken());

    garden = new int[N][M];
    isZeroHeight = new boolean[N][M];
    for(int i = 0; i < N; i++) {
        st = new StringTokenizer(br.readLine());
        for(int j = 0; j < M; j++) {
            garden[i][j] = Integer.parseInt(st.nextToken());
            isZeroHeight[i][j] = garden[i][j] == 0;
        }
    }

    // [OUTPUT]
    // K초가 지난 뒤 정원 전체 잡초 높이의 합
    int grassHeightTotal = 0;
    // 로봇이 지금까지 깎은 높이의 총합
    int robotWorkTotal = 0;

    for(int sec = 0; sec < K; sec++) {
        // 1. 높이가 1 이상인 모든 잡초 높이 1 증가 (단, 최대 높이는 9)
        for(int i = 0; i < N; i++) {
            for(int j = 0; j < M; j++) {
                garden[i][j] = garden[i][j] >= 1 && garden[i][j] < 9 ? garden[i][j] + 1 : garden[i][j];
            }
        }

        // 2. 높이가 0인 칸은 상하좌우에 높이 5 이상 잡초가 2개 이상이면 해당 칸은 높이 1로 증가(로봇 방향과 상관 없음)
        for(int i = 0; i < N; i++) {
            for(int j = 0; j < M; j++) {
                if(isZeroHeight[i][j]) {
                    int count = 0;
                    if(i + dr[0] >= 0 && garden[i + dr[0]][j] >= 5) {
                        count++;
                    }
                    if(i + dr[2] < N && garden[i + dr[2]][j] >= 5) {
                        count++;
                    }
                    if(j + dc[1] < M && garden[i][j + dc[1]] >= 5) {
                        count++;
                    }
                    if(j + dc[3] >= 0 && garden[i][j + dc[3]] >= 5) {
                        count++;
                    }

                    if(count >= 2) {
                        garden[i][j] = 1;
                        isZeroHeight[i][j] = false;
                    }
                }
            }
        }

        // 3. 로봇 행동
        // - 바라보는 방향의 앞 칸이 정원 밖이면 이동하지 않고 시계 방향으로 90도 회전
        // - 앞 칸이 정원 안이면 그 칸으로 이동하고, 그 칸의 잡초를 깎아 높이를 0으로 변경 + 깎은 높이가 7 이상이면 반시계 방향 90도 회전
        // - 방향: 0 = 북(위), 1 = 동(오른쪽), 2 = 남(아래), 3 = 서(왼쪽)
        boolean isOutside = false;
        switch(d) {
            case 0 -> isOutside = r + dr[d] < 0;
            case 2 -> isOutside = r + dr[d] >= N;
            case 1 -> isOutside = c + dc[d] >= M;
            case 3 -> isOutside = c + dc[d] < 0;
        }

        if(isOutside) {
            d = d + 1 > 3 ? 0 : d + 1;
        } else {
            switch(d) {
                case 0 -> r = r + dr[d];
                case 2 -> r = r + dr[d];
                case 1 -> c = c + dc[d];
                case 3 -> c = c + dc[d];
            }
            robotWorkTotal += garden[r][c];

            if(garden[r][c] >= 7) {
                d = d - 1 < 0 ? 3 : d - 1;
            }

            garden[r][c] = 0;
            isZeroHeight[r][c] = true;
        }

    }

    // 4. 정원 전체 잡초 높이의 합
    for(int[] row : garden) {
        for(int cell : row) {
            grassHeightTotal += cell;
        }
    }

    result.append(grassHeightTotal).append(" ").append(robotWorkTotal);
    bw.write(result.toString());

    br.close();
    bw.close();
}
```
{: file="Solution" }

***

## 💻 코드 리뷰

제출한 코드를 Claude에게 리뷰받았고 모범 답안을 요청해 비교 결과를 확인했다.

| 포인트      | 내 코드                            | 모범 답안                                                  |
| ----------- | ---------------------------------- | ---------------------------------------------------------- |
| 범위 검사   | 방향별 `switch`로 한쪽 경계만 검사 | 다음 좌표 `nr`, `nc`를 먼저 구하고 `inRange()` 하나로 검사 |
| 방향 회전   | `d + 1 > 3 ? 0 : d + 1`            | 시계 `(d + 1) % 4`, 반시계 `(d + 3) % 4`                   |
| 성장        | 삼항 연산자로 `>= 1 && < 9` 검사   | `if (h > 0) Math.min(9, h + 1)`                            |
| 빈 땅 판별  | `isZeroHeight` 배열을 따로 유지    | `garden[i][j] == 0`을 바로 읽음                            |
| 이웃 검사   | 4방향 `if` 4개                     | `for k in 0..3` + `countTall()`                            |
| 번식 동시성 | 같은 배열을 바로 수정              | 새 배열 `next`에 쓰고 마지막에 교체                        |
| 출력        | `BufferedWriter` + `close()`       | 한 줄이라 `System.out.println`                             |

### 모범 답안 전체 코드

```java
public class Answer {
    static final int[] dr = {-1, 0, 1, 0}; // 북, 동, 남, 서
    static final int[] dc = {0, 1, 0, -1};
    static int N, M;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        StringTokenizer st = new StringTokenizer(br.readLine());
        N = Integer.parseInt(st.nextToken());
        M = Integer.parseInt(st.nextToken());
        int K = Integer.parseInt(st.nextToken());

        st = new StringTokenizer(br.readLine());
        int r = Integer.parseInt(st.nextToken());
        int c = Integer.parseInt(st.nextToken());
        int d = Integer.parseInt(st.nextToken());

        int[][] garden = new int[N][M];
        for (int i = 0; i < N; i++) {
            st = new StringTokenizer(br.readLine());
            for (int j = 0; j < M; j++) {
                garden[i][j] = Integer.parseInt(st.nextToken());
            }
        }

        int cutTotal = 0;

        for (int sec = 0; sec < K; sec++) {
            // 1. 성장: 높이 1 이상인 잡초 +1 (최대 9)
            for (int i = 0; i < N; i++) {
                for (int j = 0; j < M; j++) {
                    if (garden[i][j] > 0) garden[i][j] = Math.min(9, garden[i][j] + 1); // 💡Math.min으로 삼항연산자 단순화 가능
                }
            }

            // 2. 번식: 성장이 끝난 garden은 읽기만 하고, 결과는 next에 쓴다 → "동시에" 보장
            int[][] next = new int[N][M]; // 💡실제 작업은 next에서 진행하면 동시성 확보 가능
            for (int i = 0; i < N; i++) {
                for (int j = 0; j < M; j++) {
                    next[i][j] = garden[i][j];
                    if (garden[i][j] == 0 && countTall(garden, i, j) >= 2) next[i][j] = 1;
                }
            }
            garden = next;

            // 3. 로봇 행동
            int nr = r + dr[d], nc = c + dc[d];
            if (!inRange(nr, nc)) {
                d = (d + 1) % 4;               // 정원 밖 → 시계 방향 회전
            } else {
                r = nr;
                c = nc;
                int h = garden[r][c];
                cutTotal += h;
                garden[r][c] = 0;
                if (h >= 7) d = (d + 3) % 4;   // 7 이상 깎음 → 반시계 방향 회전
            }
        }

        int heightTotal = 0;                   // 최대 2,500 × 9 = 22,500 → int로 충분
        for (int[] row : garden) {
            for (int h : row) heightTotal += h;
        }

        System.out.println(heightTotal + " " + cutTotal);
    }

    // (i, j)의 상하좌우 중 높이 5 이상인 잡초 수
    static int countTall(int[][] g, int i, int j) {
        int count = 0;
        for (int k = 0; k < 4; k++) {
            int ni = i + dr[k], nj = j + dc[k]; // 💡ni, nj가 다음 위치
            if (inRange(ni, nj) && g[ni][nj] >= 5) count++;   // 범위 검사 먼저
        }
        return count;
    }

    static boolean inRange(int r, int c) {
        return r >= 0 && r < N && c >= 0 && c < M;
    }
}
```
{: file="Answer.java" }

### 복잡도

- 시간: O(K · N · M)
  - 매초 격자를 성장·번식·합계용으로 상수 번 순회한다. 최악 1,000 × 2,500 × 상수 수준이라 여유롭다.
- 공간: O(N · M)
  - 모범 답안은 매초 `next`를 새로 만들지만, 두 배열을 번갈아 쓰면 할당 없이도 같은 효과를 낼 수 있다.

## 📜 회고

- **슈도코드 작성은 확실히 효과가 있다.**
  - 단계 별로 코드를 채운 후에 합치는 과정이 순서(성장 → 번식 → 로봇)를 이해하는데 더 도움이 되었다.
- **방향 회전은 `% 4`로 익히는게 좋겠다.**
  - 시계는 `(d + 1) % 4`, 반시계는 `(d + 3) % 4`. 음수를 피하려고 `-1` 대신 `+3`을 쓰는 게 핵심이다.
- **좌표는 다음 위치를 먼저 계산하고 범위 검사를 진행한다.**
  - 범위 검사는 별도 메소드로 사용해 중복 로직을 방지한다.
- **배열 사용 시 중복되는 의미를 가지는 배열이 존재하는지 확인한다.**
  - `isZeroHeight`는 실제로 문제 해결에 필요한 배열이 아니었다.
