### **문제**         

**레벨: S2  
알고리즘: DFS, BFS**  
**풀이시간: 1시간   
힌트 참조 유무:** 유

[https://www.acmicpc.net/problem/1260](https://www.acmicpc.net/problem/1260)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/b84ba6b8-e840-454f-8490-4bb87f00a981)

### **1 번째 시도**   

-   가장낮은 숫자부터 탐색해야 한다하니 
    -   인접행렬로 그래프 구성
-   dfs 메소드 작성법
    -   visted\[\] = true
    -   할 일
    -   for문으로 dfs 재귀호출

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.LinkedList;
import java.util.Queue;
import java.util.StringTokenizer;

public class Main {

	static StringBuilder sb = new StringBuilder();
	static boolean[] check;
	static int[][] arr;
	
	static int node, line, start;
	
	static Queue<Integer> q = new LinkedList<>();

	public static void main(String[] args) throws IOException {
		
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		
		StringTokenizer st = new StringTokenizer(br.readLine());
		node = Integer.parseInt(st.nextToken());
		line = Integer.parseInt(st.nextToken());
		start= Integer.parseInt(st.nextToken());
		
		arr = new int[node+1][node+1];
		check = new boolean[node+1];
		
		for(int i = 0 ; i < line ; i ++) {
			StringTokenizer str = new StringTokenizer(br.readLine());
			
			int a = Integer.parseInt(str.nextToken());
			int b = Integer.parseInt(str.nextToken());
			
			arr[a][b] = arr[b][a] =  1;	
		}
			//sb.append("\n");
			dfs(start);
			sb.append("\n");
			check = new boolean[node+1];
			
			bfs(start);
			
			System.out.println(sb);
		
		}
	public static void dfs(int start) {
		
		check[start] = true;
		sb.append(start + " ");
		
		for(int i = 0 ; i <= node ; i++) {
			if(arr[start][i] == 1 && !check[i])
				dfs(i);
		}
		
	}
	
	public static void bfs(int start) {
		q.add(start);
		check[start] = true;
		
		while(!q.isEmpty()) {
			
			start = q.poll();
			sb.append(start + " ");
			
			for(int i = 1 ; i <= node ; i++) {
				if(arr[start][i] == 1 && !check[i]) {
					q.add(i);
					check[i] = true;
				}
			}
		}
		
		
	}

}
```

### **배운점**

-   **DFS 작성법 연습**
