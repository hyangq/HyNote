<!--toc-->

- [拓扑排序](#拓扑排序)
	- [应用](#应用)
	- [dfs实现](#dfs实现)
	- [非递归实现 Kahn 算法](#非递归实现-kahn-算法)
		- [优化](#优化)
		- [判断是否有同向环](#判断是否有同向环)
	- [参考代码](#参考代码)

<!-- tocstop -->
# 拓扑排序
定义：拓扑排序是对有向无环图的顶点的一种排序，它使得如果存在一条从顶点A到顶点B的路径，那么在排序中B出现在A的后

## 应用
1. 定义的用法，比如选课安排什么的。
2. 判断是否存在同向环

## dfs实现
类似后序遍历，dfs 递归在归的时候，把当前的节点加入到栈中。
然后依次从栈中弹出元素，得到就是一个拓扑序列。

这样，就能所有指向当前节点的边的另一个端点都出现在当前节点之前。

```{cpp}
const int maxn = 100 + 10;
//n表示节点数,m表示边数,g保存有向边的信息,vis保存是否访问过节点,t表示当前拓扑排序的编号,topo保存拓扑排序的结果
//vis[i] = 1 表示 i 已经被访问过，vis[i] = -1 表示当前的dfs中，被访问过。
//注意,这里节点的编号是 1 ~ n
int n,m,g[maxn][maxn],vis[maxn],topo[maxn],t;
int dfs(int u){
        vis[u] = -1;
        //往下递归所有未访问过的节点
        for(int v = 1;v < n + 1;v++){
                if(g[u][v]){
                        //如果是当前流程再次访问到同一个节点，说明有环
                        if(vis[v] < 0)  return 0;

                        //如果当前节点已经被访问过（被之前的toposort访问过，排除环的情况）
						//无环的情况，存在 链的后半部分已经被访问过了的情况
						//如果仅仅判断的vis[v]的true，false，就无法区分这种情况到底是环还是后者，因此vis需要3个标记
                        if(!vis[v] && !dfs(v))  return 0;
                }
        }
        vis[u] = 1;
        topo[--t] = u;
        return 1;
}
int toposort(){
        t = n;
        memset(vis,0,sizeof(vis));
        //枚举每一个点,如果存在环（同向环）,就返回0
        for(int u = 1;u < n+1;u++){//需要遍历每个节点，因为不知道哪个节点才是根，添加vis来避免重复遍历
                if(!vis[u] && !dfs(u))
                        return 0;
        }
        return 1;
}
```

## 非递归实现 Kahn 算法
根据拓扑排序的定义，那些所有入度为0的节点都可以直接加入到最终的拓扑序列中。

1. 从当前节点选择所有入度为0的节点加入到 最终的序列中
2. 对于每个入度为0的节点，以它为起点的边的另一个端点的入度减一（就是把当前节点的信息删除）
3. 根据步骤2更新所有入度之后。重复1，2步骤，直到所有节点的入度都是0

### 优化
上面的过程，寻找度数为0的时候，每次都需要遍历一次所有的节点，这样是相当的低效的。

实际上，进行步骤2的时候，我们就可以判断度数是否为0

那么，优化之后的过程就变成了下面:

设S表示度数为0的节点的集合，T表示最终的拓扑序列

1. 枚举所有节点，把度数为0的节点加入到S中
2. 从S中取出一个节点加入到T中，然后把以这个节点为起点的边的另一个端点的入度减去1，如果减1之后度数为0，就把这个节点加入到集合S中
3. 重复步骤2，直到集合S为空

### 判断是否有同向环
当处理完所有节点之后，还存在度数大于0的节点，那么就说明有同向环。

实现的时候，可以通过计算0度节点的数目来判断是否处理完所有节点。

## 参考代码
```{cpp}
const int maxn = 1E6 + 10;
int n, d[maxn];//d是入度
vector<int> v[maxn];
void add(int from, int to) {
	++d[to];
	v[from].push_back(to);
}

void ini(int n) {
	memset(d, 0, sizeof(d));
	for (int i = 0; i <= n; ++i) {
		v[i].clear();
	}
}

int toposort() {
	queue<int> q;
	//把所有0度节点加入到队列中
	for (int i = 1; i <= n; ++i) {
		if (d[i] == 0)	q.push(i);
	}

	int cnt = q.size();
	while (!q.empty()) {
		int s = q.front(); q.pop();
		int sz = v[s].size();

		//更新所有零度节点相连的另一个节点的度数
		for (int i = 0; i < sz; ++i) {
			int t = v[s][i];
			//当出现零度节点就入队
			if (--d[t] == 0) {
				q.push(t);
				//统计出现的0度节点的个数
				++cnt;
			}
		}
	}
  //返回是否有环，如果零度节点的个数不为n，说明有同向环
	return cnt != n;
}
```
