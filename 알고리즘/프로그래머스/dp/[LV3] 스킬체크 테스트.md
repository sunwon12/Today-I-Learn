### **문제**         

**레벨: 프로그래머스 -> 스킬체크 -> 중급자   
알고리즘: DP**  
**풀이시간: 1시간  
힌트 참조 유무: 유**

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/fa7bf1d4-0e57-4158-96df-9c310d7e21d0)

### **1 번째 시도: 실패**   

**\[알고리즘 설명\]**

규칙을 찾았다. 단계를 거듭하여 만들어낼 때 **a x b x c가 가장 작은 배열 두개를 선택**해야 한다는 것이다.

그래서 아래와 같이 로직을 작성하였다.

그러나 예외를 빠드렸다. 기준을 잘못 잡은 것이었다.

배열중 앞에 있는 배열을 \[a,b\]로 잡고 뒤에 오는 배열을 \[b,c\]로 잡았다. 

문제는 뒤에오는 배열이 \[b,c\]가 앞에 있는 경우를 생각 안 한 것이었다. 

예를 들어

 \[1,2\] \[3,1\] 이렇게 되어있는 경우는 배열이 곱을 못하지만,  거꾸로 되어있을 때 \[3,1\] \[1,2\]는 사실 배열 곱이 가능한 경우의 수인 것이다.  

```
import java.util.*;

class Solution {
    public int solution(int[][] matrix_sizes) {
        int height = matrix_sizes.length;
        int ans = 0;
        
        ArrayList<int[]> list = new ArrayList<int[]>();
        for(int i = 0; i < height; i ++) {
            list.add(matrix_sizes[i]);
        }
        
        int min = Integer.MAX_VALUE;
        int[] minInd = new int[] {0,0};
        
        while(list.size() > 1) {
            
            for(int i = 0 ; i < list.size()-1; i++) {
                int first = matrix_sizes[i][0];
                int second = matrix_sizes[i][1];

                for(int j = i+1; j < list.size(); j ++) {
                   int comparingFirst = matrix_sizes[j][0];
                    int comparingSecond = matrix_sizes[j][1];

                    if(second != comparingFirst)
                        continue;
                    if(min > first*second*comparingSecond) {
                        min = first*second*comparingSecond;
                        minInd = new int[]{i,j};
                    }
                }
            }
            ans += min;
            list.add(new int[]{list.get(minInd[0])[0], list.get(minInd[1])[1]});
            list.remove(minInd[0]);
            list.remove(minInd[1]);    
        }
       
        return ans;
    }
}
```

### **2 번째 시도: 성공**   

**\[알고리즘 설명\]**

위 문제는 dp를 이용해서 풀어야 한다.

**부분 문제 정의**: dp\[i\]\[j\]는 행렬 Ai부터 Aj까지의 최소 곱셈 비용을 의미한다.

**ex)**

입력값: \[10, 20\] \[20,30\] \[30, 40\]

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/10a50b8c-9bdf-4f3b-bbdd-17c92282bb2c)

1Step: 자기 스스로 곱한것은 0이니 첫 번째는 0으로 초기화

2Step: A1과 A2를 곱할 때, A2와 A3 곱할 때 초기화 

**여기서 중요하다**

3Step: A3를 구할때는 반복문을 두번 돌아서 구한다.

dp\[0\]\[1\] + dp\[2\]\[2\] + 10\*30\*40

dp\[0\]\[0\]+ dp\[1\]\[2\] + 10\*20\*40

위 두개를 비교하는 것이다. 즉, (A1\*A2) \* A3가 작은지 A1\*(A2\*A3)가 작은지 비교해보는 것이다.

```
class Solution {
    public int solution(int[][] matrixSizes) {
        int N = matrixSizes.length;
        int[][] dp = new int[numMatrices][numMatrices];

        for (int chainLength = 2; chainLength <=N; chainLength++) { 
            for (int start = 0; start <= N - chainLength; start++) {
                int end = start + chainLength - 1;
                dp[start][end] = Integer.MAX_VALUE;
                for (int split = start; split < end; split++) {
                    int cost = dp[start][split] + dp[split + 1][end] + matrixSizes[start][0] * matrixSizes[split][1] * matrixSizes[end][1];
                    if (cost < dp[start][end]) {
                        dp[start][end] = cost;
                    }
                }
            }
        }

        return dp[0][N - 1];
    }
}
```

근데 위 로직이 무조건 성공하려면   \[10, 20\] \[20,30\] \[30, 40\] 처럼 뒤에 이따르는 배열과 곱이 가능하다는 가정이다.

 \[10, 20\]  \[30, 40\] \[20,30\] 이렇게 입력이 주어진다면 위 로직은 사용할 수 없는 것다.

프로그래머스 문제는 일단 위 로직으로 통과한다.
