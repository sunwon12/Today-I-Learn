## **#문제**         

**레벨: G5  
알고리즘:dfs**   
**풀이시간: 1시간  
힌트 참조 유무:**

**[https://www.acmicpc.net/problem/13023](https://www.acmicpc.net/problem/13023)**

![image](https://github.com/user-attachments/assets/27c78976-c816-4321-ab2e-ba4a3bed0fad)

---

## **#문제 풀이**        

이 문제에 **key point**는 dfs 경로를 다양하게 탐색할 수 있느냐이다. 보통은 그래프에서 dfs를 사용할 때 연관된 노드들을 모두 방문한다에 포커스가 맞춰져 있다. 그러나 이 문제는 **경로**에 포커스가 맞춰져있다. 그래서 우리는 dfs 함수 중 boolean\[\] = false  이 부분을 잘 봐야 한다. 이것 덕분에  이전dfs 경로에서 탐색했던 노드들도 다시 탐색하게 되는 것이다.

---

## **#풀이 코드**      

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.List;
import java.util.StringTokenizer;

public class Main {

	static boolean status;
	static List<Integer>[] list;
	static boolean[] check;
	public static void main(String[] args) throws IOException {
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		StringTokenizer st = new StringTokenizer(br.readLine());
		
		int n = Integer.parseInt(st.nextToken());
		int m = Integer.parseInt(st.nextToken());
		
		list = new ArrayList[n];
		for(int i=0; i<n; i++) {
			list[i] = new ArrayList<>();
		}
		for(int i=0; i<m; i++) {
			st = new StringTokenizer(br.readLine());
			int a = Integer.parseInt(st.nextToken());
			int b = Integer.parseInt(st.nextToken());
			list[a].add(b);
			list[b].add(a);
		}

		status = false;
		for(int i=0; i<n; i++) {
			check = new boolean[n];
			dfs(i,1);
            if(status) {
				System.out.println(1);
				return;
			}
		}
		System.out.println(0);
	}
	
	static void dfs(int idx, int depth) {
		if(depth==5) {
			status=true;
			return;
		}
		check[idx]= true;
		for(int nxt : list[idx]) {
			if(!check[nxt]) {
				dfs(nxt, depth+1);
			}
		}
        check[idx]= false; // 새로운 길에서 이전dfs 경로에서 탐색했던 노드들도 다시 탐색하게 함
	}
}
```
