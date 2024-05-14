### **문제**         

**레벨: G3  
알고리즘: 위상정렬 + DP**   
**풀이시간: 2시간  
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/1005](https://www.acmicpc.net/problem/1005)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/e6e2f935-6943-4fb7-9176-62ed0cc1fc3f)

### **1 번째 시도**  

-   입력을 받을 때마다 해당번호의 짓는 시간을 업데이트를 해줬다.
-   결과는 실패
-   이유는 입력 순서가 차례대로일거라고 가정했기 때문이다.

```
import java.util.*;
import java.io.*;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        
        int N = Integer.parseInt(br.readLine());
            
        for (int i = 0; i < N; i++) {
            String[] ss = br.readLine().split(" ");
            int bCnt = Integer.parseInt(ss[0]);
            int rCnt = Integer.parseInt(ss[1]);
            
            int[] arr = new int[bCnt];
            int[] answer = new int[bCnt]; 
            
            String[] s2 = br.readLine().split(" ");
            for (int j = 0; j < bCnt; j++) {
                arr[j] = Integer.parseInt(s2[j]);
            }
            
            answer[0] = arr[0]; 
            for (int j = 0; j < rCnt; j++) {
                String[] s3 = br.readLine().split(" ");
                int first = Integer.parseInt(s3[0]);
                int second = Integer.parseInt(s3[1]);
                int d = answer[first] + arr[second];
                answer[second] = Math.max(answer[second], d);        
            }
            int a = Integer.parseInt(br.readLine());
            System.out.println(answer[a]);
        }
    }
}
```

### **2 번째 시도** 

-   입력 순서와 상관이 없고 배열의 순서를 알려면 **위상정렬**을 써야한다는 것을 알았다

```
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.*;
 
 
public class Main {
 
    static List<Integer>[] list;
    static int n, k;
    static int[] time;
    static int[] degree;
 
    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int t = Integer.parseInt(br.readLine());
 
        for (int i = 0; i < t; i++) {
            String[] s1 = br.readLine().split(" ");
            n = Integer.parseInt(s1[0]);
            k = Integer.parseInt(s1[1]);
            time = new int[n + 1];
            degree = new int[n + 1];
            list = new ArrayList[n+1];
            String[] s3 = br.readLine().split(" ");
            for (int j = 1; j <= n; j++) {
                time[j] = Integer.parseInt(s3[j - 1]);
                list[j] = new ArrayList<>();
            }
            for (int j = 0; j < k; j++) {
                String[] s2 = br.readLine().split(" ");
                int x = Integer.parseInt(s2[0]);
                int y = Integer.parseInt(s2[1]);
                list[x].add(y);
                degree[y]++; //y건물을 짓기 위해선 x가 지어져야한다. 따라서 y에 대한 degree를 늘려준다.
            }
            int w = Integer.parseInt(br.readLine());
 
            solve(w);
        }
    }
 
    public static void solve(int target){
        Queue<Integer> queue = new ArrayDeque<>();
        int [] result = new int[n+1];
        for(int i=1; i<=n; i++){
            result[i]=time[i]; // 건물들을 짓는 비용
            if (degree[i]==0){ //제약이 없는 건물은 큐에 넣는다.
                queue.add(i);
            }
        }
 
        while (!queue.isEmpty()){
            int poll = queue.poll();
            for(Integer next: list[poll]){
                result[next]= Math.max(result[next],result[poll]+time[next]); // 방금전 지은 건물을 필요로 하는 건물들의 리스트 이므로
                degree[next]--;                                               // 전 건물을 지은 시간중에 가장 큰 값을 찾아 넣어준다.
                if(degree[next]==0){  // 건물의 조건을 만족시켰다면 큐에 넣어준다. 
                    queue.add(next);
                }
            }
        }
        System.out.println(result[target]);
    }
}
```
