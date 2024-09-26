## **#문제**         

**레벨: G2  
알고리즘: dp + 비트마스킹**   
**풀이시간: 1시간  
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/1562](https://www.acmicpc.net/problem/1562)

![image](https://github.com/user-attachments/assets/c9f75a68-d917-4b6c-9b06-44aa2c17a5be)

---

## **#문제 풀이**        

dfs를 풀어야 한다는 건 어렴풋 짐작할 수 있다. 그러나 N= 100일 때, 100 자리에 10개(0 ~ 9)를 넣는다고 생각하면 

10 ^ 100 의 작업 수가 필요하다. 당연히 이렇게 해서는 안된다. **dfs 풀이가 확고할 때는 dp 도입을 생각해본다.**

가장 루트 노드는 첫번째 자리수로서 **1 ~9** 로 시작되고 그다음부터 left와 Right로 뻗어나가면서 탐색한다. 그래프 탐색 방법에는 세가지 방식이 있는데 preorder(전위탐색), postorder(후위탐색),inorder(중위탐색)로 세 가지이다.

우린 **postOrder**로 그래프를 탐색하고 dfs를 구현한다. 참고로 ㄱpostorder는 Left - Right - Root 순으로 탐색하는 것이다.

```
solve(10,1,0000000010)
├── solve(9,0,0000000011)
│   └── solve(8,1,0000000011)
│       ├── solve(7,0,0000000011)
│       └── solve(7,2,0000000111)
└── solve(9,2,0000000110)
    ├── solve(8,1,0000000111)
    │   ├── solve(7,0,0000000111)
    │   └── solve(7,2,0000000111)  # 중복 상태
    └── solve(8,3,0000001110)
```

1.  1012 (1 -> 0 -> 1 -> 2로 시작)
2.  2102 (1 -> 2 -> 1 -> 2로 시작)

이 두 상태가 DP에서 같은 것으로 취급되는 이유는 다음과 같다.

1.  **남은 자릿수**: 둘 다 7자리가 남았다.
2.  **마지막 숫자**: 둘 다 2로 끝난다.
3.  **사용된 숫자**: 둘 다 0, 1, 2를 사용했다 (비트마스크 0000000111).

이 세 가지 정보만으로 **앞으로 만들 수 있는 유효한 계단 수의 개수**가 결정된다. 즉, 지금까지 어떤 경로로 왔는지는 중요하지 않고, 현재 상태에서 앞으로 얼마나 많은 유효한 계단 수를 만들 수 있는지가 중요하다.

---

## **#풀이 코드**      

**\[dfs 코드\]**

```
import java.io.*;

public class Main {
    static final int MOD = 1_000_000_000;            
    static final int FULL_MASK = (1 << 10) - 1;  // 비트마스킹으로 10000000000(11자리) -1 = 1111111111(10자리 / 각 자리는 9~0을 나타냄)가 되므로
    static long[][][] dp = new long[101][10][1 << 10];

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        int N = Integer.parseInt(br.readLine());

        long result = 0;
        for (int i = 1; i <= 9; i++) {
            result = (result + solve(N, i, 1 << i)) % MOD;  // 첫번째 숫자를 1~9로 시작하게 함
        }

        System.out.println(result);
    }

    static long solve(int n, int lastDigit, int usedMask) {
        if (n == 1) {
            return usedMask == FULL_MASK ? 1 : 0;
        }

        if (dp[n][lastDigit][usedMask] != 0) {
            return dp[n][lastDigit][usedMask];
        }

        long count = 0;
        if (lastDigit > 0) {
            count = (count + solve(n - 1, lastDigit - 1, usedMask | (1 << (lastDigit - 1)))) % MOD;
        }
        if (lastDigit < 9) {
            count = (count + solve(n - 1, lastDigit + 1, usedMask | (1 << (lastDigit + 1)))) % MOD;
        }

        return dp[n][lastDigit][usedMask] = count;
    }
}
```

**\[반복문 코드\]**

```
import java.util.Scanner;

public class Main {
    static final int MOD = 1000000000;
    
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int N = scanner.nextInt();
        scanner.close();
        
        long[][][] dp = new long[N + 1][10][1024];
        
        for (int i = 1; i <= 9; i++) {
            dp[1][i][1 << i] = 1;  // Set the corresponding bit for the digit 'i'
        }
        
        for (int length = 2; length <= N; length++) {
            for (int digit = 0; digit <= 9; digit++) {
                for (int mask = 0; mask < 1024; mask++) {
                    if (digit > 0) {
                        dp[length][digit][mask | (1 << digit)] += dp[length - 1][digit - 1][mask];
                        dp[length][digit][mask | (1 << digit)] %= MOD;
                    }
                    if (digit < 9) {
                        dp[length][digit][mask | (1 << digit)] += dp[length - 1][digit + 1][mask];
                        dp[length][digit][mask | (1 << digit)] %= MOD;
                    }
                }
            }
        }
        
        long result = 0;
        int fullMask = (1 << 10) - 1;  // 1111111111 in binary (all digits used)
        for (int digit = 0; digit <= 9; digit++) {
            result += dp[N][digit][fullMask];
            result %= MOD;
        }
        
        System.out.println(result);
    }
}
```
