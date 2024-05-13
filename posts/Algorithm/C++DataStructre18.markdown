---
layout: post
title:  "[자료구조/ 알고리즘] Graph"
date: 2024-05-13 23:18:04 +0900
categories: Algorithm
tags : Algorithm, Data Structure

---
---
layout: post
title: ""
description:
date: 2023-06- 12:17:04 +0900
tags :
---
## Graph

![Graph algorithms 101: popular algorithms and how to apply them](https://linkurious.com/images/uploads/2023/03/Shortest-path-algorithm.png)



### 정의 및 역사  

그래프는 가장자리로 연결된 노드 또는 정점으로 구성된 비선형 데이터 구조입니다. 그래프의 개념은 17세기에 레오하르트 오일러가 다리 연결성 문제를 연구하면서 처음 연구한 것으로 거슬러 올라갑니다. 하지만 20세기 중반이 되어서야 그래프가 컴퓨터 과학에서 대중적인 주제가 되었습니다.  

### 그래프의 특징
그래프는 비선형 자료구조로 네트워크 모델 형태를 띄는 자료구조이다. 그래프의 요소로는 아래와 같은데

1. Node (정점) : 데이터를 담은 개체
2. Edge (간선) : 노드와 노드간의 연결 정보
3. weight(가중치) : 엣지에 포함되는 가중치 (그래프에 따라 없을수도 있음)

노드는 각 노드에 따라 간선을 가질수도, 가지지 않을수도 있으며, 한 노드에서 다른 노드로 향하는 엣지가 단방향일수도 양방향일수도 있다.

### 그래프의 데이터 저장 방법  
#### Adjaceny List
연관되어 있는 노드들을 각 노드 기준으로 리스트 형태로 나타낸 것이다.

![Adjacency List meaning & definition in DSA - GeeksforGeeks](https://media.geeksforgeeks.org/wp-content/uploads/20230727155209/Graph-Representation-of-Directed-graph-to-Adjacency-List.png)
1. 인접 노드끼리 저장되므로 BFS, DFS와 같은 탐색이 자주 이루어질 때 매우 유리하다
2. 저장된 데이터 만큼만의 공간을  사용하므로 공간 효율성이 좋다

```cpp
class Graph {
	int v;
	list<int>* l;

public:
	Graph(int v) {
		this->v = v;
		l = new list<int>[v];
	}

	void addEdge(int i, int j, bool undir = true) {
		l[i].push_back(j);
		// 양방향
		if (undir) {
			l[j].push_back(i);
		}
	}

	void printAdjlist() {
		for (int i = 0; i < v; i++)
		{
			cout << i << " -- > ";
			for (int node : l[i])
			{
				cout << node << " ,";
			}
			cout << endl;
		}
	}
};
```

![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/863fa788-baef-42a7-ad25-4cfcfda2d090)

```cpp
class Node {
public:
	string name;
	list<string> nbrs;

	Node(string name) {
		this->name = name;
	}
};

class Graph {
	//중복을 허용하지 않는 맵 (비정렬)
	unordered_map<string, Node*> m;
public:
	Graph(const vector<string> & cities){
		for (const string& city : cities){
			m[city] = new Node(city);
		}
	}

	//단방향 노드에 대한 엣지 추가
	void addEdge(const string& city1, const string& city2, bool undir = false) {
		m[city1]->nbrs.push_back(city2);
		if (undir) {
			m[city2]->nbrs.push_back(city1);
		}
	}

	void printAdjList() {
		for (auto& citypair : m){
			Node* node = citypair.second;
			//range base map 루프
			for (auto it = node->nbrs.begin(); it != node->nbrs.end(); it++) {
				cout << citypair.first << " : " << *it << ", ";
			}
			delete node;
			cout << endl;
		}
	}
};

int main() {
	vector<string> cities = {"Delhi","London","Paris","New York"};
	Graph g(cities);
	g.addEdge("Delhi", "London");
	g.addEdge("New York", "London");
	g.addEdge("Delhi", "Paris");
	g.addEdge("Paris", "New York");
	g.addEdge("London", "Paris");
	g.printAdjList();
}
```

<br>
![image](https://github.com/rech4210/rech4210.github.io/assets/65288322/0009a5eb-0ade-47bc-92d0-5d577835ea1b)
<br>


Adjacency Matrix

연관되어 있는 노드들을 행렬 형태로 나타낸 것이다.  

![Different types of graphs and their corresponding adjacency matrix... |  Download Scientific Diagram](https://www.researchgate.net/publication/347300725/figure/fig1/AS:969208926044162@1608088823984/Different-types-of-graphs-and-their-corresponding-adjacency-matrix-representations-The.ppm)

1. 행렬표를 통해 노드간 간선 정보를 파악할 수 있다. (가중치의 경우 가중치로도 표시된다)
2. 공간을 더 많이 사용하게된다. (N x N)
3. 인접노드 탐색시 인접 리스트보다 탐색이 많을 수 있어 느리다.



정점 또는 에지를 기준으로 자주 조회하는 경우 인접성 목록은 인접한 정점에 상시 액세스할 수 있으므로 더 빠를 수 있습니다. 반면에 그래프의 모든 에지를 자주 반복해야 하는 경우(예: 넓이 우선 검색)에는 인접성 행렬이 더 효율적일 수 있습니다. (예로 경로 찾기 문제)


Edge List
각 Edge에 가중치가 부여되어 해당 가중치를 포함한 Edge 정보를 저장하는 리스트이다.   
```
Edges = <weight, src, dest>
```
인접한 노드들의 정보를 저장하는 대신, 각 Edge의 관계를 가중치에 따라 저장한 방식이다.

Implicit Graph
그래프의 형태를 띄고있지 않더라도 그래프로 치환하면 그래프의 형태로 나타나게 되는것을 암시적 그래프라고 한다.



-----

### BFS  

그래프 탐색 방법중 하나로 그래프의 인접 노드를 탐색하는 너비 우선 탐색이다.

```cpp
#include<iostream>
#include<list>
#include<vector>
#include<queue>
using namespace std;

//BFS Breath First Search
//인접한 노드로 탐색하기에 큐로 구현된다.

class Graph {
	int v;
	list<int>* l;
public:
	Graph(int V) {
		v = V;
		l = new list<int>[v];
	}
	void addEdge(int i, int j, bool undir = true) {
		l[i].push_back(j);
		if (undir) {
			l[j].push_back(i);
		}
	}

	void bfs(int source) {
		queue<int> q;
		bool* visited = new bool[v]{0};
		q.push(source);
		// 중복 노드 재탐색 금지
		visited[source] = true;

		while (!q.empty())
		{
			int f = q.front();
			cout << f << endl;
			q.pop();
			// source를 기준으로 해당 인덱스의 인접 노드 우선 탐색
			for (int& neighbor : l[f])
			{
				if (!visited[neighbor]) {
					q.push(neighbor);
					visited[neighbor] = true;
				}
			}
		}
	}
};

int main() {
	Graph g(7);
	g.addEdge(0,1);
	g.addEdge(0, 1);
	g.addEdge(1, 2);
	g.addEdge(2, 3);
	g.addEdge(3, 5);
	g.addEdge(5, 6);
	g.addEdge(4, 5);
	g.addEdge(0, 4);
	g.addEdge(3, 4);
	g.bfs(5);
}

result : 5, 3, 6, 4, 2, 0, 1
```

BFS는 길찾기 알고리즘에서 가장 많이 사용된다.



### DFS

깊이 우선 탐색이다

```cpp
//DFS Depth First Search
//깊이를 탐색하며 전체 그래프 탐색, 말단 노드 탐색이 주이기에 스택구조를 사용한다.

void dfs(const int source) {
	bool* visited = new bool[v] {0};
	dfsworker(source, visited);
}

//재귀적 호출에 기반하여 작성한다.
void dfsworker(const int node, bool *visited) {
	visited[node] = true;
	cout << node << ", ";

	for (const int& nbr : l[node]){
		if (!visited[nbr]) {
			dfsworker(nbr, visited);
		}
	}
}

int main() {
	Graph g(7);
	g.addEdge(0,1);
	g.addEdge(0, 1);
	g.addEdge(1, 2);
	g.addEdge(2, 3);
	g.addEdge(3, 5);
	g.addEdge(5, 6);
	g.addEdge(4, 5);
	g.addEdge(0, 4);
	g.addEdge(3, 4);
	g.dfs(1);
}

result : 1,2,4,5,3,2,6
```




### Directed Acyclic Graph (DAG)
A Directed Acyclic Graph is a directed graph with no cycles. In other words, it's a graph where there are no edges that connect vertices in such a way that you can traverse the edge and return to the starting vertex.  

각 작업이 한 번만 실행될 수 있는 작업 간의 종속성을 표현하는 데 적합한 DAG

In this example, we have created a DAG by adding edges between vertices without creating any cycle. The topological sort algorithm is used to visit all the vertices of the graph in an order that respects their dependencies (i.e., if there's an edge from A to B, then A must be visited before B).


1. **Schedule tasks**: Imagine a manufacturing process with multiple steps (e.g., cutting, drilling, painting). Topological sorting helps ensure that each step depends on previous ones being completed before moving forward.  
2. **Manage dependencies**: In software development, it's crucial to identify and resolve dependency issues between modules or libraries. Topological sort can help you detect circular dependencies and prioritize tasks accordingly.  
3. **Plan project timelines**: When planning a complex project with multiple milestones (e.g., research, design, testing), topological sorting helps ensure that each milestone is completed before moving on to the next one.


Some real-world examples of topological sort usage include:  

* Project management tools like Asana or Trello  
* Software development frameworks like Maven or Gradle  
* Manufacturing process control systems

* In finance, topological sorting can be used for portfolio optimization or risk management. For instance, an investment manager might need to prioritize investments based on their dependencies (e.g., diversification, correlation).  
* In logistics and supply chain management, it's crucial to ensure that each step in the process is completed before moving forward. Topological sorting helps optimize routes, schedules, and inventory levels.  
* Even in biology, topological sorting can be applied for understanding gene regulatory networks or protein-protein interactions.
