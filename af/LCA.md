# LCA (Lowest Common Ancestor) 최소 공통 조상
- 트리에서 두 노드가 주어졌을 때, 이 두 노드의 공통 조상 중, 가장 깊이가 깊은(가까운) 노드를 찾는 것

        1
      /   \
     2     3
    / \   / \
   4   5 6   7

노드 4와 5의 LCA는?
- 2 (두 노드의 가장 가까운 공통 조상)

노드 4와 6의 LCA는?
 1

## 방법 1: 부모 배열과 깊이(depth) 이용 (O(N) 예비작업 + O(logN) 쿼리)
- DFS를 통해 각 노드의 부모의 깊이를 기록
- 두 노드의 깊이를 맞춘 다음, 한 칸씩 부모로 올라가면서 같은 노드를 찾으면 LCA

- [코드]()

## 방법 2: Binary Lifting (O(N logN) 예비작업 + O(logN) 쿼리)
- 각 노드의 2^k번째 조상을 미리 저장

- 핵심 개념
  - up[node][k] → 노드 node의 2^k번째 조상
  - depth[node] → 노드의 깊이

- 전처리
  - DFS를 돌면서 각 노드의 부모와 깊이를 구하고,
  - up[node][k] = up[up[node][k-1]][k-1] 을 이용해
  - 모든 2^k 조상을 채워줘.

- 쿼리 처리
  - 두 노드의 깊이를 같게 맞춰줌 (큰 쪽을 log N만큼 위로 올림)
  - 공통 조상이 될 때까지 두 노드를 동시에 log N만큼 올라감
  - 마지막으로 공통 조상 바로 아래에서 한 번 더 올리면 끝!

- [코드2]()

### 방법1
```
#include <iostream>
#include <vector>

using namespace std;

const int MAX = 10001;  // 노드 최대 개수

vector<int> tree[MAX];
int parent[MAX]; // 부모 저장
int depth[MAX];  // 깊이 저장
bool visited[MAX];

// DFS로 깊이와 부모를 기록
void dfs(int node, int d) {
    visited[node] = true;
    depth[node] = d;
    for (int next : tree[node]) {
        if (!visited[next]) {
            parent[next] = node;
            dfs(next, d + 1);
        }
    }
}

// LCA 찾기
int lca(int u, int v) {
    // 깊이 맞추기
    while (depth[u] > depth[v]) u = parent[u];
    while (depth[v] > depth[u]) v = parent[v];
    // 공통 조상 찾기
    while (u != v) {
        u = parent[u];
        v = parent[v];
    }
    return u;
}

int main() {
    int n;
    cin >> n;  // 노드 개수

    for (int i = 0; i < n - 1; i++) {
        int a, b;
        cin >> a >> b;
        tree[a].push_back(b);
        tree[b].push_back(a);
    }

    dfs(1, 0); // 루트 노드는 1번이라고 가정

    int q;
    cin >> q;  // 쿼리 개수
    while (q--) {
        int u, v;
        cin >> u >> v;
        cout << lca(u, v) << '\n';
    }

    return 0;
}

```

### 방법2
```
#include <iostream>
#include <vector>
#include <cmath>

using namespace std;

const int MAX = 100001;
const int LOG = 17; // 2^17 = 131072 > 100000

vector<int> tree[MAX];
int up[MAX][LOG];  // up[node][k]: node의 2^k번째 조상
int depth[MAX];

// 전처리: DFS + Binary Lifting 초기화
void dfs(int node, int parent) {
    up[node][0] = parent;
    for (int i = 1; i < LOG; i++) {
        up[node][i] = up[up[node][i - 1]][i - 1];
    }
    for (int next : tree[node]) {
        if (next != parent) {
            depth[next] = depth[node] + 1;
            dfs(next, node);
        }
    }
}

// u와 v의 LCA를 찾는다
int lca(int u, int v) {
    if (depth[u] < depth[v]) swap(u, v);

    // 깊이 맞추기
    for (int i = LOG - 1; i >= 0; i--) {
        if (depth[u] - (1 << i) >= depth[v]) {
            u = up[u][i];
        }
    }

    if (u == v) return u;

    // 동시에 위로 올라가다가 부모가 같아지면 그 전이 LCA
    for (int i = LOG - 1; i >= 0; i--) {
        if (up[u][i] != up[v][i]) {
            u = up[u][i];
            v = up[v][i];
        }
    }

    return up[u][0];
}

int main() {
    int n;
    cin >> n;

    for (int i = 0; i < n - 1; i++) {
        int a, b;
        cin >> a >> b;
        tree[a].push_back(b);
        tree[b].push_back(a);
    }

    depth[1] = 0;
    dfs(1, 1); // 루트는 1번

    int q;
    cin >> q;
    while (q--) {
        int u, v;
        cin >> u >> v;
        cout << lca(u, v) << '\n';
    }

    return 0;
}

```
