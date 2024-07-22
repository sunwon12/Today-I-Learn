## **#문제**         

**레벨: G5  
알고리즘: dfs + cycle**  
**풀이시간: 40분  
힌트 참조 유무:**

[https://www.acmicpc.net/problem/2668](https://www.acmicpc.net/problem/2668)

![image](https://github.com/user-attachments/assets/5f5e8e8b-6f91-4d53-83c0-6bc7522a85ba)

---

## **#문제 풀이**        

처음 생각할 때 각 원소마다 존재한다 or 존재하지 않는다를 살펴서 조합을 구해야 하는 줄 알았다. 그러나 최대값인 2^의100승을 살펴야 하므로 당연히 이 방법이 아니다.

우린 조합으로 살펴보는 게 아니라 각 원소를 유심히 살펴봐야 한다.  
조합의 비밀은 숫자가 싸이클을 도는냐 아니냐이다.

예제1번의 정답이 {1,3,5\]이다. 각 원소를 살펴보자

1 > 3(num\[1\]) -> 1(num\[3\]) 

3 > 1(num\[3\]) -> 3 (num\[1\]) 

5 > 5(num\[5)  

이 과정에서 1이 조합의 원소이냐 아니냐 탐색할 때 3이 포함된 것을 알 수 있다. 당연히 3도 조합의 일부이다. 그럼 구현할 때 거치는 원소들을 미리 정답리스트에 넣을 수 있다. 1 ~ N 까지 탐색할 때 정답리스트에 들어가 있는 숫자들을 탐색 안 할 수 있다는 소리이다. 

**그럼, 거치는 원소들을 어떻게 저장할 것이냐를 생각해볼 수 있다.**  
  
1\. dfs함수 파라미터에 거친 원소들을 담은 배열을 넘길 수 있게 하기 

2\. 정답 원소는 차피 싸이클을 도므로 dfs 한 번 더 돌아 cycle 배열에 담아주기

1번 방법을 쓰게 되면 구현이 복잡해진다. dfs 파라미터에 시작노드가 무엇인지 넘겨줘야하고 dfs 구현에도 visited 배열만 쓸 수 없으므로 다른 boolean배열이 필요하다. 그럴 바에는 구현이 편한 2번 방법을 쓰는 게 낫다. **2번도 두 번 방문했는지 체크할 finished\[\] 배열이 필요하다** 

---

## **#풀이 코드**      

```
import java.io.*;
import java.util.*;

public class Main {
    static int N;
    static int[] arr;
    static boolean[] visited;
    static boolean[] finished;
    static List<Integer> cycle;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        N = Integer.parseInt(br.readLine());
        arr = new int[N + 1];
        visited = new boolean[N + 1];
        finished = new boolean[N + 1];
        cycle = new ArrayList<>();

        for (int i = 1; i <= N; i++) {
            arr[i] = Integer.parseInt(br.readLine());
        }

        for (int i = 1; i <= N; i++) {
            if (!visited[i]) {
                dfs(i);
            }
        }

        Collections.sort(cycle);
        System.out.println(cycle.size());
        for (int num : cycle) {
            System.out.println(num);
        }
    }

    static void dfs(int node) {
        visited[node] = true;
        int next = arr[node];

        if (!visited[next]) {
            dfs(next);
        } else if (!finished[next]) {
            while (next != node) {
                cycle.add(next);
                next = arr[next];
            }
            cycle.add(node);
        }

        finished[node] = true;
    }
}
```

---

## **#비슷한 유형**     

**백준 9466 텀 프로젝트**

[https://jsw5913.tistory.com/171](https://jsw5913.tistory.com/171)

 [\[백준 9466\] 텀 프로젝트 / 자바 / dfs

#문제         레벨: G3알고리즘: dfs 풀이시간: 1시간힌트 참조 유무:https://www.acmicpc.net/problem/9466#문제 풀이        본 유형은 나에게는 신유형이다. 보통 dfs라면 cycle를 돌지 않고 한 번 간 곳

jsw5913.tistory.com](https://jsw5913.tistory.com/171)
