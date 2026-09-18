---
title: 코딩 테스트
date: 2026-09-18

tags: [Algorithm, DataStructure, CodingTest, Cpp, Python]
description: "코딩 테스트 대비를 위한 언어 선택 가이드부터 주요 자료구조와 알고리즘 학습, 그리고 효율적인 코드 작성을 위한 최적화 및 변수 관리 전략을 다룹니다."

category: study
thumbnail: ""
draft: false
---

# How to Start

매 스터디마다 주제가 정해집니다.

해당 주제에 대한 설명을 들은 후, 같이 몇 문제를 풉니다.
주제의 난이도와 스터디원의 수준에 따라 코딩을 할 수도 있고 개념적으로만 풀고 넘어갈 수도 있습니다.

스터디가 끝난 후 같이 풀었던 문제와 추가 문제가 연습에 올릴 것입니다.
이 문제를 다음 스터디 전까지 푸시면 됩니다.

피어 리뷰를 하시는 분은 다른 스터디원의 코드를 분석하여 개선해야할 점이 있다면 메모해 오시면 됩니다.

## 문제 푸는 방법

## 연습문제풀기

문제는 반드시 공개 혹은 맞았을 때 공개 둘 중 하나로 해야합니다. `코드 리뷰를 위함`
이미 풀었던 문제여도 연습이 올라오고 난 이후 재제출해야 연습 현황이 업데이트 됩니다.

## 문제 제한 조건

코드의 최적화를 늘 염두에 두고 코딩해야합니다.
모든 입력을 미리 입력받지 않고 문제를 풀기위해 **최소한으로 필요한 입력만을 받으며** 출력합니다.
또한 디버깅과 코드리뷰를 위해 변수명을 **빠르게 이해되고, 실수 안 나게** 적어주세요!

- `n, m, k` → 크기
- `arr, graph, dp` → 자료구조
- `cur, next` → 흐름
- `ans, cnt` → 결과

## 도전 과제

변수의 최소화에 도전하세요. 동시에 존재하는 코드의 수가 적으면 됩니다. 예를 들어

```python
for(int i=0;i<10;i++)
    sum+=i;
for(int j=1;j<10;j++)
    mul*=j
```

위 코드의 경우 `i`, `j` 두 변수가 존재하지만 동시에 존재하는 변수의 수는 1개입니다.
이런 식으로 동시에 존재하는 변수의 수를 확인합니다.
- 길이가 `n`인 배열이 있다면 그 배열의 길이만큼 변수의 수로 칩니다.
- `int` 2개를 붙여 하나의 `long long`을 만들어도 변수 2개로 간주합니다.

**의미론적인 변수의 수**를 측정하여 이 수를 줄이는 것을 목표로 합니다.
변수의 수를 최소화하기위하여 변수들간의 상관관계를 고민해야 할 것입니다.

그리고 이를 통해 나중에 계산 도중에 도출되는 중간 값을 저장하기 위해 변수가 몇 개 필요한지 코딩하면서 바로 알 수 있습니다.


# How to Select Language

언어는 흔히 C, C++, JAVA, Python 중에 하나를 배우면 됩니다. 아래는 각 언어에 대한 대략적인 설명입니다.

## C

다른 언어에 비해 이미 구현 되어 있는 것이 적습니다.
C언어만 제출 가능한 삼성 알고리즘 테스트 B,C형을 제외하면 C언어로 코딩 테스트를 보면 매우 불리합니다.
단 코딩 실력을 높이기 위해 C로 구현하는 것을 연습하셔도 좋습니다.

## C++

JAVA와 Python에 비해 빠릅니다.
C++로 알고리즘을 풀기를 추천합니다.
단 기존 구현되어 있는 method나 function을 쓸 때 시간 복잡도를 반드시 고려해야합니다.

## JAVA

느립니다. 몇몇 회사는 코딩테스트시 JAVA를 지원하지 않을 수도 있습니다.
구현되어 있는 기능의 양은 C++과 비슷합니다.
C++이 도저히 익숙해지지 않는다면 대안이 될 수 있습니다.

## Python

대부분의 회사에서 지원하지 않습니다.
구현되어있는 기능이 강력하여 다른 언어에서는 코딩해야하는 것이 이미 함수로 구현되어 있을 수 있습니다.
드물지만 특정 코딩테스트에서는 python이 필수일 수도 있습니다.

# Common STL (Beginner)

## vector

동적 배열이다. vector뒤에 원소를 넣고 빼는 연산에 특화되어 있다.
for 문에 쓸때는 iterator를 쓰면 편하다.
`>>`쓸 때 `>`과 `>` 사이에 스페이스가 있어야 한다. 안그러면 비트연산 `>>`으로 해석해서 컴파일 에러가 생길 수도 있다.

```cpp
#include<cstdio>
#include<vector>

using namespace std;

vector<int> vt;//int외의 다른 것들도 가능. double,long long, struct 혹은 vector마저도 가능.

int main(void){
    int sizeOfArr,tmp;
    scanf("%d",&sizeOfArr);//input의 갯수
    for(int i=0;i<sizeOfArr;i++){
        scanf("%d",&tmp);
        vt.push_back(tmp);//vector에 저장(제일 뒤에 저장됨)
    }
    for(vector<int>::iterator iter=vt.begin();iter!=vt.end();iter++)
    //iterator가 end에 도달하면 이미 모든 원소 탐색을 한 것이다.
        printf("%d ",*iter);//iter앞에 *이 붙어있는 점에 주의하자
    return 0;
}
```

## queue

이름에서부터 알 수 있듯이 queue이다. FIFO 연산에 특화되어 있다.

```cpp
#include<cstdio>
#include<queue>

using namespace std;

queue<int> qu;

int main(void){
    int sizeOfArr,tmp;
    scanf("%d",&sizeOfArr);//input의 갯수
    for(int i=0;i<sizeOfArr;i++){
        scanf("%d",&tmp);
        qu.push(tmp);//queue에 저장(제일 뒤에 저장됨)
    }
    while(!qu.empty()){//큐가 빌때까지
        printf("%d ",qu.front());//제일 앞에 있는 원소 출력
        qu.pop();//앞에 있는 원소 제거 pop의 리턴 값이 원소가 아님에 주의
    }
    return 0;
}
```

## pair

2개의 원소를 합쳐 하나의 원소로 나타낸다.
pair가 강력해지는 부분은 **비교연산**이다.
우선적으로 앞의 원소를 비교하고 그 후에 뒤의 원소를 비교한다.

예를 들어
```cpp
pair<int, pair<double, float> >
```

위 자료형을 가지는 원소 $a$, $b$가 있다고 해보자. $a$의 int를 $a_1$, double을 $a_2$, float을 $a_3$ 그리고 이에 상응하게 b의 원소 $b_1$,$b_2$,$b_3$라고 부르자.

그러면 우선 $a_1$과 $b_1$을 비교하여 같지 않으면 그 결과를 리턴한다.
즉 $a_1 < b_1$ 이면 $a < b$, $a_1 > b_1$ 이면 $a > b$ 이다. $a_1$과 $b_1$이 같으면 pair를 비교한다.

그런데 pair 연산은 왼쪽 원소부터 비교를 시작하므로 double을 비교하기 시작한다.
결국 $a_2$와 $b_2$를 비교하여 같지 않으면 그 결과를 리턴한다.

$a_2$와 $b_2$가 같으면 $a_3$와 $b_3$를 비교하며 이 마저도 같을 때 같다라는 연산이 성립한다.

```cpp
#include<cstdio>
#include<vector>
#include<utility>//pair있는 header

using namespace std;

vector<pair<int,int> > vt;//vector나 queue등에 넣을 수 있음.

int main(void){
    int a1,a2;
    for(int i=0;i<10;i++){
        scanf("%d %d",&a1,&a2);
        vt.push_back(make_pair(a1,a2));//make_pair로 pair를 만듦
    }
    for(vector<pair<int,int> >::iterator iter=vt.begin();iter!=vt.end();iter++)
        printf("%d %d\n",iter->first,iter->second);//첫번째 원소는 first 두번째 원소는 second로 접근. vector의 iterator로 접근해서 포인터 접근함.
    return 0;
}
```

## sort

말 그대로 정렬한다. 시간복잡도는  $O(n\log n)$ 이다. 기본적으로 **오름차순**으로 정렬된다. sort함수에 다른 인자를 넣어 내림차순을 구현할 수도 있지만 코딩테스트에서는 보통 원소에 `-`를 곱해 내림차순을 구현한다.

```cpp
#include<cstdio>
#include<vector>
#include<algorithm>//sort있는 header

using namespace std;

vector<int> vt;

int main(void){
    int sizeOfArr,tmp;
    scanf("%d",&sizeOfArr);//input의 갯수
    for(int i=0;i<sizeOfArr;i++){
        scanf("%d",&tmp);
        vt.push_back(tmp);
    }
    sort(vt.begin(),vt.end());//vector vt 정렬
    for(vector<int>::iterator iter=vt.begin();iter!=vt.end();iter++)//iterator가 end에 도달하면 이미 모든 원소 탐색을 한 것이다.
        printf("%d ",*iter);//iter앞에 *이 붙어있는 점에 주의하자
    return 0;
}
```

## map

Key와 value를 mapping해주는 STL이다.
Key가 0부터 연속된 양의 정수라면 배열과 다를 바 없다.

그러나 문제를 풀다보면 문자열끼리 대응시키거나 정수가 아닌 자료형과 mapping 시켜야 하는 경우가 있다. 그럴때 이 STL을 쓰면 편하다.

```cpp
#include<cstdio>
#include<map>
#include<string>

using namespace std;

map<string,double> studentScore;

int main(void){
    int sizeOfInput;
    char str_c[50];
    scanf("%d",&sizeOfInput);//input의 갯수
    for(int i=0;i<sizeOfInput;i++){
        double score;
        scanf("%s %lf",str_c,&score);
        string str = str_c;
        studentScore.insert(make_pair(str,score));//map에 넣을때는 insert를 사용해도 되고
        //studentScore[str]=score;//대괄호[]를 이용하여 배열처럼 사용할 수도 있습니다.
    }
    scanf("%s",str_c);
    string str = str_c;
    if(studentScore.find(str)==studentScore.end())//Key가 들어온 적이 없는지 확인하기 위해 이렇게 접근합니다.
        printf("Failed to find\n");
    else
        printf("Score is %lf\n",studentScore[str]);//무조건 들어왔다는 확신이 있다면 find한 값이 end()와 같은지 비교하지 않고 바로 대괄호로 접근해도 됩니다.
    return 0;
}
```

## set

수학에서 집합은 중복 원소를 허용하지 않는다.
이와 유사하게 set STL은 중복된 원소가 들어올 시 무시한다.
**특정 원소의 존재 여부**를 확인할 때 사용하기 좋은 STL이다.

```cpp
#include<cstdio>
#include<set>
#include<string>

using namespace std;

set<string> studentName;

int main(void){
    int sizeOfInput;
    char str_c[50];
    scanf("%d",&sizeOfInput);//input의 갯수
    for(int i=0;i<sizeOfInput;i++){
        scanf("%s",str_c);
        string str = str_c;
        studentName.insert(str);//set에 넣을때는 insert를 사용하면 됩니다.
    }
    scanf("%s",str_c);
    string str = str_c;
    if(studentName.find(str)==studentName.end())//set에 원소가 존재하는지 확인하기 위해 이렇게 접근합니다.
        printf("Failed to find\n");
    else
        printf("Success to find\n");
    return 0;
}
```

## stack

FIFO 연산에 특화되어 있는 STL입니다. 잘 생각해보면 vector STL을 stack처럼 사용할 수 있기 때문에 상대적으로 적게 쓰입니다.

```cpp
#include<cstdio>
#include<set>
#include<string>

using namespace std;

set<string> studentName;

int main(void){
    int sizeOfInput;
    char str_c[50];
    scanf("%d",&sizeOfInput);//input의 갯수
    for(int i=0;i<sizeOfInput;i++){
        scanf("%s",str_c);
        string str = str_c;
        studentName.insert(str);//set에 넣을때는 insert를 사용하면 됩니다.
    }
    scanf("%s",str_c);
    string str = str_c;
    if(studentName.find(str)==studentName.end())//set에 원소가 존재하는지 확인하기 위해 이렇게 접근합니다.
        printf("Failed to find\n");
    else
        printf("Success to find\n");
    return 0;
}
```

## tuple

pair를 확장하여 다수의 원소를 합쳐 하나의 원소로 나타낸다. C++11부터 지원한다.

```cpp
#include<cstdio>
#include<tuple>
#include<vector>

using namespace std;

vector<tuple<int,int,int> > coordinate;

int main(void){
    for(int i=0;i<10;i++){
        int x,y,z;
        scanf("%d %d %d",&x,&y,&z);
        coordinate.push_back(make_tuple(x,y,z));//make_tuple로 tuple를 만듦
    }
    for(vector<tuple<int,int,int> >::iterator iter=coordinate.begin();iter!=coordinate.end();iter++)
        printf("%d %d %d\n",get<0>(*iter),get<1>(*iter),get<2>(*iter));//get을 사용하여 원소에 접근
    return 0;
}
```

## priority_queue

우선순위 queue이다. 비교연산을 통해 가장 큰 값부터 반환한다.

```cpp
#include<cstdio>
#include<queue>

using namespace std;

priority_queue<int> qu;

int main(void){
    int sizeOfArr,tmp;
    scanf("%d",&sizeOfArr);//input의 갯수
    for(int i=0;i<sizeOfArr;i++){
        scanf("%d",&tmp);
        qu.push(tmp);//queue에 저장
    }
    while(!qu.empty()){//큐가 빌때까지
        printf("%d ",qu.top());//제일 큰 원소 출력
        qu.pop();//앞에 있는 원소 제거 pop의 리턴 값이 원소가 아님에 주의
    }
    return 0;
}
```

## binary_search

이분 탐색할때 사용한다. 시간 복잡도는  $O(\log n)$ 이다. **호출하기 전에 정렬되어있어야한다.**

```cpp
#include<cstdio>
#include<vector>
#include<algorithm>//binary_search있는 header

using namespace std;

vector<int> vt;

int main(void){
    int sizeOfArr,tmp;
    scanf("%d",&sizeOfArr);//input의 갯수
    for(int i=0;i<sizeOfArr;i++){
        scanf("%d",&tmp);
        vt.push_back(tmp);
    }
    sort(vt.begin(),vt.end());//vector vt 정렬
    for(int i=0;i<sizeOfArr;i++){
        scanf("%d",&tmp);
        if(binary_search(vt.begin(),vt.end(),tmp))
            printf("Exists\n");
        else
            printf("There is no such element\n");
    }
    return 0;
}
```

## Graph

STL은 아니지만 엄청 많이 사용할 자료구조니 기억하자.
코테를 준비하는 거면 아래 두 가지 구현 방식만 기억해도 충분하다.

**인접 행렬(Adjacency Matrix):**
- 그래프에서 정점 간의 연결 관계를 2차원 배열로 나타내는 방식이다.
- 정점의 수가 $V$개 일 때 $V \times V$ 크기의 행렬을 사용해서 간선(Edge)의 존재 여부를 0(없음)과 1(있음)으로 나타낸다.

```cpp
vector<vector<int>> graph;
vector<vector<pair<int,int>>> graph;
// 이것만 알고있으면 DFS·BFS·Dijkstra·Topology Sort 대부분 커버될거다.
```

# Common Algorithm (Beginner)

초급반의 최종 목표는 일반적인 코딩 테스트에 나오는 알고리즘 문제를 전부 풀 수 있는 정도에 도달하는 것입니다. 따라서 배울 알고리즘은 최신 코딩 테스트를 참조하여 선정했습니다.

## Common Algorithm

- Brute-force
- DFS
- BFS
- DP
- Greedy
- Binary search

## Rare Algorithm

- Minimum spanning tree
- Trie
- Disjoint Set
- Segment tree

## 알고리즘은 아니지만 연습해야하는 것

- Simulation
- Recursion

## Bruth-force

입력 크기가 작거나 (보통 $N \le 10-20$), 모든 경우를 확인해야 할 때 쓴다.
가능한 모든 경우를 다 본다고 생각하면 된다.
보통 문제를 볼 때 해법이 떠오르기 전 `어? 무식하지만 이러면 될거 같은데..` 가 대부분 Bruth-force다.

```cpp
// ex) 모든 부분집합 탐색
#include <bits/stdc++.h>
using namespace std;

int n = 3;
vector<int> arr = {1, 2, 3};

void solve() {
    for (int mask = 0; mask < (1 << n); mask++) {
        vector<int> subset;
        for (int i = 0; i < n; i++) {
            if (mask & (1 << i)) {
                subset.push_back(arr[i]);
            }
        }

        for (int x : subset) cout << x << " ";
        cout << "\n";
    }
}
```

## DFS(Depth-First Search)

그래프 탐색, 모든 경로 탐색, 백트래킹을 할 때 쓴다.
**끝까지 파고 들어가서 찾는 방식**이라고 생각하면 된다.

```cpp
vector<vector<int>> graph;
vector<bool> visited;

void dfs(int cur) {
    visited[cur] = true;
    cout << cur << " ";

    for (int next : graph[cur]) {
        if (!visited[next]) {
            dfs(next);
        }
    }
}
```

## BFS(Breadth-First Search)

최단 거리(가중치 없음) 또는 레벨 탐색에 쓰인다.
**가까운 것부터 확장해서 찾는 방식**이라고 생각하면 된다.
대부분의 경우 queue로 구현한다.

```cpp
vector<vector<int>> graph;
vector<int> dist;

void bfs(int start) {
    queue<int> q;
    q.push(start);
    dist[start] = 0;

    while (!q.empty()) {
        int cur = q.front(); q.pop();

        for (int next : graph[cur]) {
            if (dist[next] == -1) {
                dist[next] = dist[cur] + 1;
                q.push(next);
            }
        }
    }
}
```

## DP(Dynamic Programming)

중복되는 계산이 많거나, 최적값(최대·최소·경우의 수)을 찾아야 하는 문제면 DP를 먼저 떠올려보자.
문제를 동일한 로직의 작은 문제로 쪼개서 결과를 저장하고 그걸 재사용 하는 방식이다.

```cpp
// ex1) 피보나치 수열
vector<int> dp(100, -1);

int fib(int n) {
    if (n <= 1) return n;
    if (dp[n] != -1) return dp[n];

    return dp[n] = fib(n-1) + fib(n-2);
}

// ex2) 반복문 형식
vector<int> dp(100);

void solve() {
    dp[0] = 0;
    dp[1] = 1;

    for (int i = 2; i <= 10; i++) {
        dp[i] = dp[i-1] + dp[i-2];
    }
}
```

## Greedy

지금 최선의 선택이 전체 최적을 보장할 때 사용한다.
매 순간 선택의 갈래가 올 때마다 이게 최선의 선택인거 같은데.. 라는 생각이 들면 된다.
수학적으로 증명하는 방식도 있기는 한데 간단한 Greedy라면 추천하지만 문제가 복잡해지면 답이 없으니 많이 풀면서 체득하자.

```cpp
// ex) 동전 문제: 동전을 최소화 할 때 무조건 큰 금액의 동전부터 쓰는 게 최선이다.
vector<int> coins = {500, 100, 50, 10};
int n = 1260;
int cnt = 0;

void solve() {
    for (int coin : coins) {
        cnt += n / coin;
        n %= coin;
    }
}
```

## Binary Search

앞서 언급했든이 정렬된(sorted) 배열 또는 조건을 만족하는 최소/최대를 찾을 때 쓴다.
어떤 값을 찾을 때 도저히 시간 내로 못 찾을 것 같으면 무조건 Binary Search부터 의심하고 가면 옳다.
하지만 웬만하면 대기업 코테에서 자주 등장하지는 않는거 같달까

```cpp
int binary_search(vector<int>& arr, int target) {
    int left = 0, right = arr.size() - 1;

    while (left <= right) {
        int mid = (left + right) / 2;

        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }

    return -1;
}
```

# Note

## DFS Logic

사실 dfs 로직은 항상 정해져 있다고 보고 기계적으로 풀면 된다.
추가 로직은 유형에 따라 넣을 수도 있고, 안 넣을수도 있는데 다음과 같이 정해진 구간이있다.

```cpp
void dfs(int curr) {
    visited[curr] = 1;              // (A) 들어오자마자

    for (int next : adj[curr]) {    // (B) 인접 노드 순회
        if (!visited[next]) {
            dfs(next);              // (C) 내려가기
        }
    }

    // (D) 돌아오면서 처리 (optional)
}
```

| 문제 유형  | DFS 안에 추가되는 것  |
| ------ | -------------- |
| 연결 요소  | 없음             |
| 경로 찾기  | parent 저장      |
| 사이클 탐지 | 부모 체크          |
| 트리     | depth, subtree |
| 백트래킹   | 상태 복구          |

```cpp
// 1. Depth / 거리 계산
int depth[MAX];

void dfs(int curr) {
    visited[curr] = 1;

    for (int next : adj[curr]) {
        if (!visited[next]) {
            depth[next] = depth[curr] + 1;  // ⭐ 여기!
            dfs(next);
        }
    }
}

// 2. 부모 노드 저장 (트리 문제들)
int parent[MAX];

void dfs(int curr) {
    visited[curr] = 1;

    for (int next : adj[curr]) {
        if (!visited[next]) {
            parent[next] = curr;   // ⭐ 여기!
            dfs(next);
        }
    }
}

// 3. Subtree 크기 계산 (트리 DP)
int subtree[MAX];

void dfs(int curr) {
    visited[curr] = 1;
    subtree[curr] = 1;  // 자기 자신

    for (int next : adj[curr]) {
        if (!visited[next]) {
            dfs(next);
            subtree[curr] += subtree[next];  // ⭐ 여기!
        }
    }
}

// 4. 사이클 탐지 (부모 체크)
bool hasCycle = false;

void dfs(int curr, int parent) {
    visited[curr] = 1;

    for (int next : adj[curr]) {
        if (!visited[next]) {
            dfs(next, curr);
        }
        else if (next != parent) {  // ⭐ 여기!
            hasCycle = true;
        }
    }
}

// 5. 경로 저장 (백트레킹)
vector<int> path;

void dfs(int curr) {
    visited[curr] = 1;
    path.push_back(curr);   // ⭐ 들어갈 때

    for (int next : adj[curr]) {
        if (!visited[next]) {
            dfs(next);
        }
    }

    path.pop_back();        // ⭐ 돌아올 때
}

// 6. 특정 조건 만족 개수 세기
int cnt = 0;

void dfs(int curr) {
    visited[curr] = 1;

    if (adj[curr].size() == 1) cnt++;  // ⭐ leaf 체크

    for (int next : adj[curr]) {
        if (!visited[next]) {
            dfs(next);
        }
    }
}
```


## String Parsing: `stringstream`

**stringstream**
| 문자열을 입출력 스트림처럼 다루는 객체다. `cin`, `cout`처럼 사용 가능하다.

```cpp
#include <sstream> // sstream 라이브러리를 필요로 한다.

stringstream ss("10 20");
int a, b;
ss >> a >> b;  // 10, 20
```

같은 stringstream 변수 재사용 시 `clear()`과 `str("")` 초기화를 동시에 수행해줘야 한다는 걸 주의하자.
```cpp
ss.clear();      // 상태 초기화
ss.str("30 40"); // 버퍼 교체 혹은 초기화
```

문자열 파싱할 때 많이..많이 편하다.
sstream과 getline을 활용하면 된다.
```cpp
// 1. 기본적인 문자열 파싱 방법
string s = "10+20+30";
stringstream ss(s);
string token;

while (getline(ss, token, '+')) {
    // token: "10", "20", "30"
}

// 2. 형변환도 다음과 같이 stoi를 사용하지 않고도 할 수 있다.
// 문자열 → 숫자  
int x = stoi("123");  
  
// stringstream 방식  
stringstream ss("123");  
int x;  
ss >> x;

// 숫자 → 문자열
stringstream ss;
ss << 123;
string s = ss.str();
```

## String Operations

```cpp
#include <iostream>
#include <string>

// 1. 문자열 접근
string s;
s[0];   // 빠르지만 범위 체크 로직이 없어 위험하다.
s.at(); // 범위를 체크하고 예외를 띄워주기에 안전하다.

// 2. 문자열 결함 (Concatenation)
string a, b;
string c = a + b;

// 3. 길이 및 비어있는지 확인
s.lenght();
s.size(); // length와 동일하다.
s.empty();

// 4. 문자열 검색
int pos = s.find("target") // 해당 문자열이 시작하는 위치를 반환한다.
if (s.find("target") == string:: npos) // 못 찾을 시 string::npos 반환

// 5. 부분 문자열
s.substr(start, length);

// 6. 문자열 수정
string s = "hello";
s.insert(5, " world"); // "hello world"
s.erase(0, 5); // " world"
string s = "hello world";
s.replace(0, 5, "hi"); // "hi world"

// 7. 문자열 비교
string a = "abc";
string b = "abd";
if (a < b) // 사전순 비교, 문자열 길이가 같을 때 사용하자.

// 8. 숫자 <-> 문자열 변환
string s = to_string(x);
int x = stoi(s);  
long l = stol(s);  
double d = stod(s);

// 9. 커스텀 정렬
sort(vec.begin(), vec.end(), [](string a, string b) {
    return a + b > b + a;
});

// 10. 문자 변환 (대소문자)
#include <cctype>

char c = 'a';
toupper(c); // 'A'
tolower(c); // 'a'
```

```cpp
#include <iostream>
#include <string>

// 1. realloc + copy
string s;
for (int i = 0; i < 10000; i++) {
	s += "a";
}

// 2. realloc X & copy X (Efficient)
string s;
s.reserve(10000); // reserve를 통해 처음에 한 번만 메모리를 할당한다.

for (int i = 0; i < 10000; i++) {
	s += "a";
}
```

## Simulation

거의 대부분 아래와 같다는 걸 유념하고, 객체화, 함수화를 잘 해서 풀자.
```
시간이 흐른다
→ 객체가 움직인다
→ 충돌한다
→ 상태가 변한다
→ 반복
```

아래는 실전용 템플릿이다.
```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>

using namespace std;

#define FASTIO ios::sync_with_stdio(false); cin.tie(nullptr);

const int MAX = 105;

int n, m, T;

int board[MAX][MAX];

int dy[4] = {-1, 0, 1, 0};
int dx[4] = {0, 1, 0, -1};

struct Robot {
    int y;
    int x;
    int dir;
    int energy;
    bool alive;
};

vector<Robot> robots;

bool inRange(int y, int x) {
    return 0 <= y && y < n && 0 <= x && x < m;
}

void input() {

    cin >> n >> m >> T;

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cin >> board[i][j];
        }
    }
}

void moveRobots() {

    for (auto &r : robots) {

        if (!r.alive)
            continue;

        int ny = r.y + dy[r.dir];
        int nx = r.x + dx[r.dir];

        // 벽 충돌
        if (!inRange(ny, nx)) {

            r.dir = (r.dir + 2) % 4;

            continue;
        }

        r.y = ny;
        r.x = nx;
    }
}

void processCollision() {

    // 예시:
    // 같은 칸에 있는 로봇 처리

}

void updateState() {

    // 체력 감소
    // 제거
    // 점수 계산
}

void simulate() {

    for (int t = 0; t < T; t++) {

        moveRobots();

        processCollision();

        updateState();
    }
}

void printAnswer() {

}

int main() {

    FASTIO;

    input();

    simulate();

    printAnswer();
}
```