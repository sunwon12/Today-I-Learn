## **1\. 문제**

---

**레벨: G4  
알고리즘: dfs(백트래킹)**  
**풀이시간: 30분  
힌트 참조 유무: 유**

**[https://www.acmicpc.net/problem/9663](https://www.acmicpc.net/problem/9663)**

![image](https://github.com/user-attachments/assets/9651e83f-5caa-490e-a45a-090f11c052cd)

## **2\. 문제 풀이**

---

## **실패 시도:**

-   서로 공격하지 않는 경우의 수로 구하려고 하였으나 N^2 C N 의 시간 복잡도가 나왔습니다. 이는 한 눈에 봐도 큰 경우의 수 입니다. 아무리 백트래킹을 한다그래도 N = 15, 시간제한 = 10초 안에 불가능할 거라 판단하였습니다.
-   **_전체 경우의 수 - 서로 공격하는 경우_** 로 구하려고 하였으나 서로 공격하는 경우에서 2 ~ N이 서로 공격하는 경우를 구할 때 중복이 발생함으로 중복을 해결하는 과정이 너무 복잡해보였습니다.  <--- **잘못된 방법인 냄새**

### **실패 원인: N^2 C N에 대한 오해**

**N^2 C N**은 "N^2개의 칸에서 N개의 퀸을 선택하여 배치하는 경우의 수"를 의미합니다. 즉, 단순히 **N개의 퀸을 N^2개의 칸에 놓는 방법**을 계산하는 것입니다. 이는 퀸을 체스판의 모든 칸에 놓는 경우의 수를 세는 방식(**브루트 포스)**으로, **퀸들이 서로 공격할 수 없는지**는 고려하지 않은 계산입니다. 퀸들이 서로 공격할 수 없는지(**백트래킹**)를 고려해서 최악의 시간 복잡도를 생각해보면 **O(N!)**입니다.

### **올바른 풀이:**

**이차원 배열로 체스판을 구현할 수 있겠지만, 퀸의 특성인 직선, 대각선만 살펴보면 되므로 체스판을 일차원 배열로 구현하였습니다. 이 말이 이해가 안 가시면 코드에서 possible 함수를 보시면 됩니다.**

**일차원 배열의 인덱스는 행을 나타내고 요소의 값은 열을 나타냅니다.**

## **3\. 풀이 코드**

---

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.StringTokenizer;

public class Main {

	static int[] arr;
	static int N;
	static int de = 0;
	
	public static void main(String[] args) throws IOException {
		
		BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
		String str = br.readLine();
		
		N = Integer.parseInt(str);
		
		arr = new int[N];
		
		dfs(0);
		System.out.println(de);
}
	public static void dfs(int depth) {
		
		if(depth == N) {
			de++;
			return;
		}
		
		for(int i = 0 ; i < N; i++) {
			arr[depth] = i;
			if(possible(depth)) { // 백트래킹
				dfs(depth+1);
			}
		}	
	}
	
	public static boolean possible(int col) {
		
		for(int i = 0 ; i < col ; i++) {
		//행에 일치하는게 있는지 판별
		if(arr[i]==arr[col]) {
			return false;
		}
		//대각선에 일치하는게 있는지 판별
         //두 점의 x의 차이와 y의 차이가 같다면 대각선에 일치하는 것이므로
		else if(Math.abs(col-i) == Math.abs(arr[col]-arr[i])) {
			return false;
		}
			
		}
		
		return true;
	}
}
```
