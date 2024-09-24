## **#문제**         

**레벨: G1  
알고리즘: 이분탐색**   
**풀이시간: 1시간  
힌트 참조 유무: 유**

[https://www.acmicpc.net/problem/1300](https://www.acmicpc.net/problem/1300)

![image](https://github.com/user-attachments/assets/c48bafa4-b51d-41ef-8df6-93ecb6bd86fd)

---

## **#문제 풀이**        

1 ~ N\*N의 숫자들을 이분탐색하여 알아낼 것이다. **해당숫자(L포인터와 R포인트에 의한 mid)**를 행의 값으로 나눠주면 그 행에서 **해당숫자**보다 작거나 같은 숫자의 개수가 나오는 것이다. 각행마다 이 과정을 실행하고 더한 값이 B배열의 위치값(인덱스)이다**.**  **이렇게 나온 값이 K보다 작거나 크다면 왼쪽 포인터와 오른쪽 포인터를 조절해서 해당숫자(mid)를 다시 만든다.**
​
예시를 들어 설명하겠다.
​
N=4이고 K=9일 때
​
1.  초기 범위: left = 1, right = N \* N = 4 \* 4 = 16
2.  이진 탐색을 시작한다:
    -   mid = (1 + 16) / 2 = 8이다
    -   countLessOrEqual(8)을 계산한다: 1행: min(8/1, 4) = 4, 2행: min(8/2, 4) = 4, 3행: min(8/3, 4) = 2, 4행: min(8/4, 4) = 2, 총합: 4 + 4 + 2 + 2 = 12이다
    -   12 >= 9이므로, answer = 8, right = 7로 설정한다
    -   mid = (1 + 7) / 2 = 4이다
    -   countLessOrEqual(4)를 계산한다: 4 + 2 + 1 + 1 = 8이다
    -   8 < 9이므로, left = 5로 설정한다
    -   mid = (5 + 7) / 2 = 6이다
    -   countLessOrEqual(6)을 계산한다: 4 + 3 + 2 + 1 = 10이다
    -   10 >= 9이므로, answer = 6, right = 5로 설정한다
    -   left > right이므로 루프를 종료한다
3.  최종 결과는 answer = 6이다
​
따라서 이 알고리즘에 따르면 N=4, K=9일 때 출력값은 6이다.
​
이는 4x4 행렬에서 9번째로 작은 수가 6이라는 의미다. 실제로 4x4 행렬을 만들어보면:
​
1 2 3 4 2 4 6 8 3 6 9 12 4 8 12 16
​
이 행렬에서 9번째로 작은 수가 6임을 확인할 수 있다.
​

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
