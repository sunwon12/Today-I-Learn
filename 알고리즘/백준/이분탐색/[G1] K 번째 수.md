## **#문제**         

**레벨: G1  
알고리즘: 이분탐색**   
**풀이시간: 1시간  
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/1300](https://www.acmicpc.net/problem/1300)

![image](https://github.com/user-attachments/assets/c48bafa4-b51d-41ef-8df6-93ecb6bd86fd)

---

## **#문제 풀이**        

1 ~ N\*N이 일차원 배열에 오름차순으로 정렬되어 있다고 상상하자. 우리는 이분탐색으로 값을 찾아낼 것이다. (1+N\*N)/2의 값부터 살펴볼 것이다. 원래 같으면 (1+N\*N)/2와 배열의 값을 비교해서 왼쪽 포인터와 오른쪽 포인터를 조절할 것이다. **그러나 배열의 값을 다 배열에 넣으면 10만 곱하기 10만이므로 이미 시간초과이다.** **그래서 우리는 배열에 숫자를 넣었다 치고 계산하는 거다.** 그걸 어떻게 하느냐? (1+N\*N)/2를 행의 값으로 나눠주면 그 행에서 **해당숫자**( (1+N\*N)/2)보다 작거나 같은 숫자의 개수가 나오는 것이다. 만약 **해당숫자가 너무 커 그 행에 존재하는 개수(N)보다 큰 경우 N을 더해주는 것이다.   
이렇게 모든 행을 탐색해서 더해준 값이 K보다 작거나 크다면 왼쪽 포인터와 오른쪽 포인터를 조절해서 해당숫자를 다시 만든다.**

---

## **#풀이 코드**      

```
import java.io.*;
import java.util.*;

public class Main {
    static long N, k;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        N = Long.parseLong(br.readLine());
        k = Long.parseLong(br.readLine());

        long left = 1;
        long right = N * N;
        long answer = 0;

        while (left <= right) {
            long mid = (left + right) / 2;
            long count = countLessOrEqual(mid);

            if (count >= k) {
                answer = mid;
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }

        System.out.println(answer);
    }

    static long countLessOrEqual(long x) {
        long count = 0;
        for (int i = 1; i <= N; i++) {
            count += Math.min(x / i, N);
        }
        return count;
    }
}
```
