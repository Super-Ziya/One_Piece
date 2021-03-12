### Dijkstra 算法

---

> 单源最短路径算法，用于计算一个节点到其他所有节点的最短路径
>
> 特点：以起始点为中心向外层层扩展，直到扩展到终点为止
>
> 核心思想：贪心算法
>
> 前提条件：图中不存在负权边

- 问题描述：在无向图 G=(V,E) 中，假设每条边 E[i] 的长度为 w[i]，找到由顶点 V0 到其余各点的最短路径（单源最短路径）

- 算法描述

  - 从选定点开始抛入优先队列（路径一般为 0），`boolean` 数组标记 0 的位置（最短为 0），0 周围连通的点抛入优先队列中，并把各点距离记录到对应数组内（如果小于就更新，大于就不动，初始第一次是无穷肯定会更新），第一次结束
  - 从队列中抛出距离最近的点 `B`（第一次是 0 周围邻居），标记这个点为 `true`，并将这个点的邻居加入队列（下一次确定的最短点在前面未确定和这个点邻居中产生），更新通过 `B` 点计算各个位置的长度，小于则更新

  <img src="C:\Users\13085\Desktop\git_work\算法\image\Dijkstra算法1.png" alt="Dijkstra算法1" style="zoom: 50%;" />
  - 重复 2 操作，直到所有点确定

  <img src="C:\Users\13085\Desktop\git_work\算法\image\Dijkstra算法2.png" alt="Dijkstra算法2" style="zoom:50%;" />

- 实现

```java
public class dijkstra {
	static class node{
		int x;	//节点编号
		int length;	//长度
        
		public node(int x,int lenth){
			this.x = x;
			this.length = length;
		}
	}

	public static void main(String[] args) {
		int[][] map = new int[6][6];	//记录权值，顺便记录链接情况，可以考虑附加邻接表
		initmap(map);	//初始化
		boolean bool[] = new boolean[6];	//判断是否已经确定
		int len[] = new int[6];	//长度
        
		for(int i=0;i<6;i++){
			len[i] = Integer.MAX_VALUE;
		}
		Queue<node> q1 = new PriorityQueue<node>(com);
		len[0] = 0;	//从0这个点开始
		q1.add(new node(0,0));
		int count = 0;	//计算执行了几次dijkstra
		while(!q1.isEmpty()) {
			node t1 = q1.poll();
			int index = t1.x;	//节点编号
			int length = t1.length;	//节点当前点距离
			bool[index] = true;	//抛出的点确定
			count++;	//执行6次就可确定不需继续执行
			for(int i=0;i<map[index].length;i++){
				if(map[index][i] > 0 && !bool[i]){
					node node = new node(i,length+map[index][i]);
					if(len[i] > node.length){	//需要更新节点的时候更新节点并加入队列
						len[i] = node.length;
						q1.add(node);
					}
				}
			}
		}
		for(int i=0;i<6;i++){
			System.out.println(len[i]);
		}
	}
    
	static Comparator<node> com = new Comparator<node>(){
		public int compare(node o1,node o2) {
			return o1.length - o2.length;
		}
	};

	private static void initmap(int[][] map) {
		map[0][1] = 2; map[0][2] = 3; map[0][3] = 6;
		map[1][0] = 2; map[1][4] = 4; map[1][5] = 6;
		map[2][0] = 3; map[2][3] = 2;
		map[3][0] = 6; map[3][2] = 2; map[3][4] = 1; map[3][5] = 3;
		map[4][1] = 4; map[4][3] = 1;
		map[5][1] = 6; map[5][3] = 3;	
	}
}
```

- 复杂度
  - 时间复杂度 O(N2)

