## Kruskal 算法

> 最小生成树算法：基于贪心的思想，首先把所有的边按照权值先从小到大排列，接着按照顺序选取每条边，如果这条边的两个端点不属于同一集合，那么就将它们合并，直到所有的点都属于同一个集合为止。合并到一个集合用到并查集

- 流程：
  - 将图 G 看做一个森林，每个顶点为一棵独立的树
  - 将所有边加入集合 S，一开始 S = E
  - 从S中拿出一条最短的边 (u,v)，如果 (u,v) 不在同一棵树内，则连接 u,v 合并这两棵树，同时将 (u,v) 加入生成树的边集 E'
  - 重复 (3) 直到所有点属于同一棵树，边集 E' 就是一棵最小生成树
- 实现

```java
public void kruskal() {
    int index = 0;                      // rets数组的索引
    int[] vends = new int[mEdgNum];     // 用于保存"已有最小生成树"中每个顶点在该最小树中的终点。
    EData[] rets = new EData[mEdgNum];  // 结果数组，保存kruskal最小生成树的边
    EData[] edges;                      // 图对应的所有边

    edges = getEdges();	// 获取"图中所有的边"
    sortEdges(edges, mEdgNum);	// 将边按照"权"的大小进行排序(从小到大)

    for (int i=0;i<mEdgNum;i++) {
        int p1 = getPosition(edges[i].start);      // 获取第i条边的"起点"的序号
        int p2 = getPosition(edges[i].end);        // 获取第i条边的"终点"的序号
        int m = getEnd(vends, p1);                 // 获取p1在"已有的最小生成树"中的终点
        int n = getEnd(vends, p2);                 // 获取p2在"已有的最小生成树"中的终点
        
        if (m != n) {	//"边i"与"已经添加到最小生成树中的顶点"没有形成环路
            vends[m] = n;                       // 设置m在"已有的最小生成树"中的终点为n
            rets[index++] = edges[i];           // 保存结果
        }
    }
    // 统计并打印"kruskal最小生成树"的信息
    int length = 0;
    for (int i=0;i<index;i++){
        length += rets[i].weight;
    }
    System.out.printf("Kruskal=%d: ", length);
    for (int i=0;i<index;i++){
        System.out.printf("(%c,%c) ", rets[i].start, rets[i].end);
    }
    System.out.printf("\n");
}
```

- 时间复杂度
  - 先对边按权值从小到大排序，这一步的时间复杂度为为 O(ElogE)
  - 使用并查集，最坏的情况可能要枚举完所有的边，此时要循环 |E| 次，这一步的时间复杂度为 O(Eα(V))，其中 α 为 Ackermann 函数，其增长非常慢，可以视为常数
  - Kruskal 算法的时间复杂度为 O(ElogE)