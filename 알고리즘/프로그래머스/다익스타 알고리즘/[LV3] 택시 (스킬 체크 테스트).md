### **문제**         

**레벨: Lv3  
알고리즘: 다익스트라 알고리즘**   
**풀이시간: 1시간  
힌트 참조 유무: 유**

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/24ebd4a9-4063-4c81-8310-69d02c512e15)

  
  

### **1 번째 시도**   

**\[알고리즘 설명\]**

다익스트라 알고리즘으로 S를 기준으로, A를 기준으로, B를 기준으로 각각의 노드의 최소거리르 구해야 한다.

그리하여

출발점 -> A,B가 찢어지는 노드 -> 찢어지고 A가 가는 거리, 찢어지고 B가 가는 거리

출발점 -> A가 가는 거리 + 출발점 -> B가 가는 거리 

두 가지 경우를 다 고려할 수 있다. 

```
import java.util.*;

class Solution {
    public int solution(int n, int s, int a, int b, int[][] fares) {
        List<List<int[]>> graph = new ArrayList<>();
        for (int i = 0; i <= n; i++) {
            graph.add(new ArrayList<>());
        }
        
        for (int[] fare : fares) {
            int u = fare[0];
            int v = fare[1];
            int w = fare[2];
            graph.get(u).add(new int[]{v, w});
            graph.get(v).add(new int[]{u, w});
        }
        
        int[] distFromS = dijkstra(s, n, graph);
        int[] distFromA = dijkstra(a, n, graph);
        int[] distFromB = dijkstra(b, n, graph);
        
        int minFare = Integer.MAX_VALUE;
        for (int k = 1; k <= n; k++) {
            if (distFromS[k] != Integer.MAX_VALUE && distFromA[k] != Integer.MAX_VALUE && distFromB[k] != Integer.MAX_VALUE) {
                int fare = distFromS[k] + distFromA[k] + distFromB[k];
                minFare = Math.min(minFare, fare);
            }
        }
        
        return minFare;
    }
    
    private int[] dijkstra(int start, int n, List<List<int[]>> graph) {
        int[] dist = new int[n + 1];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[start] = 0;
        PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[1]));
        pq.add(new int[]{start, 0});
        
        while (!pq.isEmpty()) {
            int[] current = pq.poll();
            int u = current[0];
            int currentDist = current[1];
            
            if (currentDist > dist[u]) continue;
            
            for (int[] neighbor : graph.get(u)) {
                int v = neighbor[0];
                int weight = neighbor[1];
                if (dist[u] + weight < dist[v]) {
                    dist[v] = dist[u] + weight;
                    pq.add(new int[]{v, dist[v]});
                }
            }
        }
        
        return dist;
    }
   
}
```
