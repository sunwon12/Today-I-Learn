
## **#문제**         
​
**레벨: G1  
알고리즘: 수학(포함배제원리 + 모듈러 연산)**   
**풀이시간: 1시간  
힌트 참조 유무: 유**
​
[https://www.acmicpc.net/problem/16565](https://www.acmicpc.net/problem/16565)
​
![image](https://github.com/user-attachments/assets/f303ca74-732b-4a93-a313-71336629f8ac)
​
---
​
## **#문제 풀이**        
​
쌍이 맞는 조합 카드 4개를 뽑고 나서 나머지 카드를 뽑으면 된다. 
​
쌍이 맞는 4개의 카드를 뽑는 경우의 수는 13이고 나머지 카드를 뽑는 건 52-4= 48에서 N-4를 뽑으면 되니
​
48C(N-4)를 해주면된다. 13x 48C(N-4)
​

**위는 틀렸다.**
​

위 공식은 두 개 이상의 포카드가 있는 경우를 **중복 계산**할 수 있다. 예를 들면 N=8일 때 K 조합을 뽑고  48C4에서 포카드 조합이 나오거나 안 나오거나 두 케이스를 커버칠 수 있는 거 아니야? 라고 생각할 수 있다. 그 생각이 맞다. 그러나 우리는 중복을 해결하지 못했다. 

1\. 처음에 조합카드를 뽑을 때 K조합을 뽑고 남은 카드 중 무작위로 4장을 뽑을 때 에이스 조합이 나왔다.

2\. 처음에 조합카드를 뽑을 때 에이스 조합을 뽑고 남은 카드 중 무작위로 4장을 뽑을 때 K 조합이 나왔다.

이 두가지 케이스가 같다. 즉,  13x 48C(N-4)에는 중복이 발생하는 공식이다.
​
---
​
우린 조합이 1일 때, 조합이 2일 떄 .... 조합이 13일 때를 케이스 별로 나눠서 구해야 한다.
​
여기서 배워가는 게 **포함배제 원리, 모듈러 연산**이다.
​
모듈러 연산은 아래 링크에서 학습하고 오자.
​
[https://jsw5913.tistory.com/204](https://jsw5913.tistory.com/204)
​
​
-   포함-배제의 원리를 써서 1개 이상의 포카드 세트를 포함하는 모든 경우를 계산한다.
-   각 단계:
    -   i개의 포카드 세트를 고르는 방법의 수를 계산한다 (13Ci).
    -   남은 카드를 고르는 방법의 수를 계산한다 ((52-4i)C(N-4i)).
    -   이 두 값을 곱한다.
-   홀수 개의 세트를 고르는 경우는 더하고, 짝수 개의 세트를 고르는 경우는 뺀다 (포함-배제 원리).
​
---
​
## **#풀이 코드**      
​
```
import java.util.*;
​
public class Main {
    static final int MOD = 10007;
​
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int N = sc.nextInt();
        System.out.println(solve(N));
        sc.close();
    }
​
    static int solve(int N) {
        if (N < 4) return 0;
        
        long total = 0;
        int maxSets = Math.min(N / 4, 13);  // 최대 가능한 포카드 세트 수
​
        for (int i = 1; i <= maxSets; i++) {
            long ways = combination(13, i);  // i개의 포카드 세트 선택
            ways = (ways * combination(52 - 4*i, N - 4*i)) % MOD;  // 나머지 카드 선택
            if (i % 2 == 1) {
                total = (total + ways) % MOD;  // 홀수 개의 세트: 더하기
            } else {
                total = (total - ways + MOD) % MOD;  // 짝수 개의 세트: 빼기
            }
        }
​
        return (int)total;
    }
​
    static long combination(int n, int r) {
        if (r > n) return 0;
        long result = 1;
        r = Math.min(r, n - r);
        for (int i = 0; i < r; i++) {
            result = (result * (n - i)) % MOD;
            result = (result * modInverse(i + 1, MOD)) % MOD;
        }
        return result;
    }
​
    static long modInverse(int a, int m) {
        return power(a, m - 2, m);
    }
​
    
    static long power(long base, int exponent, int mod) {
        long result = 1;
        base %= mod;
        //제곱 계산 수를 줄여주기 위함
        while (exponent > 0) {
            if (exponent % 2 == 1) {
                result = (result * base) % mod;
            }
            exponent >>= 1;
            base = (base * base) % mod;
        }
        return result;
    }
}
```
