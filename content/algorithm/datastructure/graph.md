+++
title  = "graph"
toc    = true
weight = 5
+++

## Intro
- Topological Sort

#### Minmum Spanning Tree
  - Prim
  - Kruskal

#### Graph Traversal
- Depth First Search
- Breadth First Search
- Best Firset Search
- Hamiltionian Path
- Eulerian Path

#### Shortest Path
- Floyd-Warshal
- Dijkstra
- Bellman-Ford

## Implementation
### Adjust Matrix

### Adjust List
```
// A simple representation of graph using STL
#include<bits/stdc++.h>
using namespace std;
 
// A utility function to add an edge in an
// undirected graph.
void addEdge(vector<int> adj[], int u, int v)
{
    adj[u].push_back(v);
    adj[v].push_back(u);
}
 
// A utility function to print the adjacency list
// representation of graph
void printGraph(vector<int> adj[], int V)
{
    for (int v = 0; v < V; ++v)
    {
        cout << "\n Adjacency list of vertex "
             << v << "\n head ";
        for (auto x : adj[v])
           cout << "-> " << x;
        printf("\n");
    }
}
 
// Driver code
int main()
{
    int V = 5;
    vector<int> adj[V];
    addEdge(adj, 0, 1);
    addEdge(adj, 0, 4);
    addEdge(adj, 1, 2);
    addEdge(adj, 1, 3);
    addEdge(adj, 1, 4);
    addEdge(adj, 2, 3);
    addEdge(adj, 3, 4);
    printGraph(adj, V);
    return 0;
}
```


```
#include <cstdio>
#include <vector>
#include <list>
#include <utility>
 
using namespace std;
 
int main()
{
    int vertices, edges, v1, v2, weight;
     
    printf("Enter the Number of Vertices -\n");
    scanf("%d", &vertices);
     
    printf("Enter the Number of Edges -\n");
    scanf("%d", &edges);
     
    // Adjacency List is a vector of list.
    // Where each element is a pair<int, int>
    // pair.first -> the edge's destination
    // pair.second -> edge's weight
    vector< list< pair<int, int> > > adjacencyList(vertices + 1);
     
    printf("Enter the Edges V1 -> V2, of weight W\n");
     
    for (int i = 1; i <= edges; ++i) {
        scanf("%d%d%d", &v1, &v2, &weight);
         
        // Adding Edge to the Directed Graph
        adjacencyList[v1].push_back(make_pair(v2, weight));
    }
     
    printf("\nThe Adjacency List-\n");
    // Printing Adjacency List
    for (int i = 1; i < adjacencyList.size(); ++i) {
        printf("adjacencyList[%d] ", i);
         
        list< pair<int, int> >::iterator itr = adjacencyList[i].begin();
         
        while (itr != adjacencyList[i].end()) {
            printf(" -> %d(%d)", (*itr).first, (*itr).second);
            ++itr;
        }
        printf("\n");
    }
     
    return 0;
}
```
