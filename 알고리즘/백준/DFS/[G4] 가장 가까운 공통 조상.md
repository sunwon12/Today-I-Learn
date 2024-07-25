## **#문제**         

**레벨: G4  
알고리즘: dfs + LCA**   
**풀이시간: 50분  
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/3584](https://www.acmicpc.net/problem/3584)

![image](https://github.com/user-attachments/assets/bb168b67-04b2-462f-afd2-4f4bbec7ecf8)

---

## **#문제 풀이**        

LCA(Lowest Common Ancestor) 문제에서 풀이법은 다양하다. 

 **첫 번째 방법**\\

1\. dfs를 통해 각 노드에 깊이를 매겨준다.

2\. 9와 7의 공통조상을 찾으려면 둘의 깊이가 맞을 때까지 한 쪽이 올라간다.

3\. 깊이가 맞다면 두 숫자가 동시에 한 칸 씩 올라간다.

4\. 결국 8을 찾게 된다.

![image](https://github.com/user-attachments/assets/ca49bd00-1f9a-4c94-b9f6-5f795fcf34ec)

**두 번째 방법**

**15와 11의 공통조상을 찾는다고 가정하자**

**1\. 15가 루트노드를 만날 때까지 visited\[\]를 체크하며 올라간다.**

**2\. 그 후 11은 visited\[\] = true로 되어 있는 노드를 만날 때 까지 올라간다. visited\[\] = true로 되어있는게 공통조상이다.**

![image](https://github.com/user-attachments/assets/0b3e8a21-6c11-48b3-b940-c5ff68e93cfc)

**개인적으로 두 번째 방법이 간단한 풀이법 같다.** 

---

## **#풀이 코드**      

#### **1번째 풀이**

```
import java.util.*;
import java.io.*;

public class Main {
    static ArrayList<Integer>[] tree;
    static int[] parent;
    static int[] depth;
    static boolean[] visited;
    static int root;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int T = Integer.parseInt(br.readLine());

        for (int t = 0; t < T; t++) {
            int N = Integer.parseInt(br.readLine());
            
            tree = new ArrayList[N + 1];
            parent = new int[N + 1];
            depth = new int[N + 1];
            visited = new boolean[N + 1];

            for (int i = 1; i <= N; i++) {
                tree[i] = new ArrayList<>();
            }

            // 루트 노드 찾기
            boolean[] isChild = new boolean[N + 1];
            
            for (int i = 0; i < N - 1; i++) {
                StringTokenizer st = new StringTokenizer(br.readLine());
                int a = Integer.parseInt(st.nextToken());
                int b = Integer.parseInt(st.nextToken());
                tree[a].add(b);
                isChild[b] = true;
            }

            // 루트 찾기
            for (int i = 1; i <= N; i++) {
                if (!isChild[i]) {
                    root = i;
                    break;
                }
            }

            dfs(root, 0, 0);

            StringTokenizer st = new StringTokenizer(br.readLine());
            int node1 = Integer.parseInt(st.nextToken());
            int node2 = Integer.parseInt(st.nextToken());

            System.out.println(lca(node1, node2));
        }
    }

    static void dfs(int node, int par, int dep) {
        visited[node] = true;
        parent[node] = par;
        depth[node] = dep;

        for (int child : tree[node]) {
            if (!visited[child]) {
                dfs(child, node, dep + 1);
            }
        }
    }

    static int lca(int a, int b) {
        while (depth[a] != depth[b]) {
            if (depth[a] > depth[b]) {
                a = parent[a];
            } else {
                b = parent[b];
            }
        }

        while (a != b) {
            a = parent[a];
            b = parent[b];
        }

        return a;
    }
}
```

**2번째 풀이**

```
import java.util.*;
import java.io.*;

public class Main {
    static int[] parent;
    static boolean[] visited;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int T = Integer.parseInt(br.readLine());

        for (int t = 0; t < T; t++) {
            int N = Integer.parseInt(br.readLine());
            
            parent = new int[N + 1];
            visited = new boolean[N + 1];

            for (int i = 0; i < N - 1; i++) {
                StringTokenizer st = new StringTokenizer(br.readLine());
                int p = Integer.parseInt(st.nextToken());
                int c = Integer.parseInt(st.nextToken());
                parent[c] = p;
            }

            StringTokenizer st = new StringTokenizer(br.readLine());
            int node1 = Integer.parseInt(st.nextToken());
            int node2 = Integer.parseInt(st.nextToken());

            System.out.println(findLCA(node1, node2));
        }
    }

    static int findLCA(int a, int b) {
        while (a != 0) {            // 한 숫자가 루트 노드까지 탐색한다
            visited[a] = true;
            a = parent[a];
        }
        
        while (b != 0) {            // 다른 숫자가 visited = true를 만날 때까지 탐색한다.
            if (visited[b]) {
                return b;
            }
            b = parent[b];
        }
        
        return -1; // 못 찾는 경우
    }
}
```
