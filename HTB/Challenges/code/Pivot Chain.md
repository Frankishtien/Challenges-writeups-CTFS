# Pivot Chain



```c++

#include <bits/stdc++.h>
using namespace std;

using ll = long long;
const ll INF = LLONG_MAX;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int N, M;
    cin >> N >> M;

    string start, target;
    cin >> start >> target;

    unordered_map<string, int> id;
    vector<vector<pair<int, ll>>> adj(N);

    int cnt = 0;

    auto getId = [&](const string& s) {
        if (!id.count(s)) {
            id[s] = cnt++;
        }
        return id[s];
    };

    int src = getId(start);
    int dest = getId(target);

    vector<tuple<string, string, ll>> edges;

    for (int i = 0; i < M; i++) {
        string u, v;
        ll w;
        cin >> u >> v >> w;
        edges.push_back({u, v, w});

        getId(u);
        getId(v);
    }

    for (auto &[u, v, w] : edges) {
        adj[id[u]].push_back({id[v], w});
    }

    vector<ll> dist(N, INF);

    priority_queue<
        pair<ll, int>,
        vector<pair<ll, int>>,
        greater<pair<ll, int>>
    > pq;

    dist[src] = 0;
    pq.push({0, src});

    while (!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();

        if (d != dist[u])
            continue;

        if (u == dest)
            break;

        for (auto &[v, w] : adj[u]) {
            if (dist[v] > d + w) {
                dist[v] = d + w;
                pq.push({dist[v], v});
            }
        }
    }

    if (dist[dest] == INF)
        cout << -1 << '\n';
    else
        cout << dist[dest] << '\n';

    return 0;
}

```

<img width="1917" height="899" alt="image" src="https://github.com/user-attachments/assets/295f2300-8dc5-41f4-84a6-9385c1cd4d5c" />

















