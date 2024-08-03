## **#문제**         

**레벨: G2  
알고리즘: dfs + dp**  
**풀이시간: 1시간   
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/1103](https://www.acmicpc.net/problem/1103)

![image](https://github.com/user-attachments/assets/11839c13-f81e-4479-9e87-da95d233b978)

---

## **#문제 풀이**        

dfs+dp 코드 구현은 워낙 다양하다. 그래서 자기에게 맞는 코드를 찾아야 한다. 자기가 이해하기 쉬운 코드를 짜야한다. visited배열이 언제 true,false가 되고 dp 값이 어떻게 언제 채워지는지 정확히 이해해야 한다.  
  
아래 코드는 한 번씩 이동할 때마다 dp배열에 +1씩 새긴다. 만약 dp 배열에 값이 지금 새겨넣으려는 값보다 큰 경우 그 경로에서는 탐색을 멈춘다. 왜냐하면 이미 이전에 깊이 탐색했다는 뜻으로. vsited배열은 무한루프를 탐색하기 위함이기 때문에 다른경로에 영향을 주면 안된다. 한 경로에서 visited=true인 곳을 방문했을 때 무한루프로 처리한다. 

그러면 의문이 들 수 있다. 한경로에서 true로 표시했다가 한 경로 탐색이 끝나면 false로 하면 다른 경로에서도 이전에 탐색했던 인덱스 위치를 탐색할 수 있지 않냐? 그렇다. 만약 시간이 넉넉했다면 문제될 게 없다. 그런데 문제에서는 시간초과가 뜬다. 그래서 우리는 dp배열을 쓰는 것이다. 이전에 탐색했을 때 해당인덱스의 dp값이 더 큰경우는 더 깊이 탐색했던 경우이므로 탐색을 중단한다. 이렇게 visited배열로 탐색을 중단시키는 게 아니라, dp배열로 탐색을 중단시키는 것이다.

  
**다른 경로에서 부분이라도 겹치면 안될 때    ====================> visited배열 도입**

**다른 경로에서 부분겹쳐도 되고 한 경로에서는 방문했던 곳을 방문하면 안되고 시간초과 우려가 있을 때 =======> dp배열 도입**

---

## **#풀이 코드**      

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {

	static int n,m;
	static int hole = -99;
	static int max=-1;
	static boolean flag= false;
	static int[][] map,dp;
	static boolean[][] visited;
	static int[] dx = {1,-1,0,0};
	static int[] dy = {0,0, 1,-1};
	
	public static void main(String[] args) throws IOException{
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		StringTokenizer st = new StringTokenizer(br.readLine());
		n = Integer.parseInt(st.nextToken());
		m = Integer.parseInt(st.nextToken());
		
		map = new int[n][m];
		dp =  new int[n][m];
		visited = new boolean[n][m];
		
		for(int i=0; i<n; i++) {
			String[] line = br.readLine().split("");
			for(int j=0; j<line.length; j++) {
				if(line[j].equals("H")) {
					map[i][j] = hole;
				}else {
					int num = Integer.parseInt(line[j]);
					map[i][j] = num;
				}
			}
		}

		visited[0][0] = true;
		dfs(0,0,1);
		
		if(flag) {
			System.out.println(-1);
		}else {
			System.out.println(max);
		}
	}
	
	static void dfs(int x, int y, int cnt) {
		if(cnt>max) {
			max = cnt;
		}
		dp[x][y] = cnt;
		
		for(int i=0; i<4; i++) {
			int move = map[x][y];
			int nx = x+ (move*dx[i]);
			int ny = y+ (move*dy[i]);
			
			if(nx<0 || ny<0 || nx>n-1 || ny>m-1 || map[nx][ny] == hole) {
				continue;
			}
			
			// 이미 방문한 곳을 다시 한번 방문하면 무한루프로 리턴 
			if(visited[nx][ny]) {
				flag = true;
				return;
			}
			
			// 이미 깊이 탐색한 부분 스킵
			if(dp[nx][ny] > cnt) continue;
			
			visited[nx][ny]= true;
			dfs(nx, ny, cnt+1);	
			visited[nx][ny]= false;
		}
		
	}
}
```
