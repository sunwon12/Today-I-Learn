## **#문제**         

**레벨: G5  
알고리즘: 구현 + 자료구조**  
**풀이시간: 1시간  
힌트 참조 유무:**

**[https://www.acmicpc.net/problem/14891](https://www.acmicpc.net/problem/14891)**
![image](https://github.com/user-attachments/assets/27bf5d3d-796b-4911-8307-c59182bb325d)


---

## **#문제 풀이**        

위 문제는 간단한 구현 문제이다. 회전을 하기 전 그 상태가 모든 톱니바퀴의 회전 여부를 결정하기 때문이다. 만약 처음 그상태가 아니라 톱니바퀴 하나를 돌리고 나서 다른 톱니바퀴의 회전 여부를 결정해야 했다면 이것보다는 조금 더 구현이 복잡해졌을 수 있다. 

1\. 해당 톱니바퀴를 기준으로 연속적으로 왼쪽 톱니바퀴들이 회전하는지 체크하고 같은 방식으로 오른쪽 톱니바퀴들이 회전하는지 체크한다.

2\. 회전 시켜준다. 

3\. 스코어를 구한다. 

**난 이 문제를 통해 구현연습을 한 것도 있지만 자료구조를 더 집중적으로 공부해봤다.** 

톱니바퀴의 N,S 상태를 한줄로 1과0의 연속으로 한줄로 주었다. 우린 습관적으로 이를 배열로 받고 회전할 때는 temp 방식을 이용해 숫자를 한 칸식 밀어 회전을 구현할 수 있다. 하지만, 톱니바퀴의 회전을 구현할 때 용이한 자료구조가 있지 않는가? 앞, 뒤에서 입출력을 전부 할 수 있는 **DeQue**(Double Ended Queue / **LinkedList로 구현**)가 있다. 시계방향으로 회전한다고 하면, 맨 마지막 요소를 추출해 맨 앞 자리에 추가하면 끝이다. **즉, 배열로 회전을 구현했다면 무조건 배열을 한번 돌아야 하니 O(n)이다. Deque는 O(1)이다.**

**그러나 두 개의 톱니바퀴의 N,S를 비교할 때는 배열이 더 효율적이다.** 배열은 arr\[2\]로 한 번에 접근할 수 있고 Deque는 중간 요소에 한 번에 접근할 수 없어 인덱스 0부터 2까지 탐색 후 인덱스 2를 반환한다. **즉, 배열은 시간복잡도가 O(1)이고, Deque는 O(n)이다. 그렇다 하면 우린 어떤 자료구조가 효율적일까?**  

현재 톱니바퀴의 톱니는 8개이다. 8은 상당히 작은 숫자이다. 배열은 연속적으로 메모리가 할당되어 캐시친화적이다. 또한 JVM의 배열 최적하로 인해 **배열을 사용하는 것이 제일 빠르다.**

+**) ArrayDeque로 Deque구현  
****ArrayDeque로 Deque구현하면 회전 구현 때 Deque의 이점을 가져갈 수 있고 요소를 접근할 때는 배열처럼 한 번에 접근할 수 있어. LinkedList로 구현하는 것보다 더 효율적이다. 그렇다 하더라도 톱니가 적을 때는 배열이 더 효율적이다.**

**우린 개발을 할 때 이런 자료구조의 장단점을 생각하며 자료구조를 선택해 개발해야한다.**

**[https://jsw5913.tistory.com/73](https://jsw5913.tistory.com/73)**



---

## **#풀이 코드**      

**배열 자료구조 이용**

```
import java.util.*;
import java.io.*;

public class Main {
    static int[][] gears = new int[4][8];
    static int K;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        // 톱니바퀴 상태 입력
        for (int i = 0; i < 4; i++) {
            String line = br.readLine();
            for (int j = 0; j < 8; j++) {
                gears[i][j] = line.charAt(j) - '0';
            }
        }

        // 회전 횟수 입력
        K = Integer.parseInt(br.readLine());

        // 회전 명령 처리
        for (int i = 0; i < K; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            int gearNum = Integer.parseInt(st.nextToken()) - 1;
            int direction = Integer.parseInt(st.nextToken());

            rotate(gearNum, direction);
        }

        // 점수 계산
        int score = 0;
        for (int i = 0; i < 4; i++) {
            if (gears[i][0] == 1) {
                score += (1 << i);
            }
        }

        System.out.println(score);
    }

    static void rotate(int gearNum, int direction) {
        int[] rotations = new int[4];
        rotations[gearNum] = direction;

        // 왼쪽 톱니바퀴 확인
        for (int i = gearNum - 1; i >= 0; i--) {
            if (gears[i][2] != gears[i+1][6]) {
                rotations[i] = -rotations[i+1];
            } else {
                break;
            }
        }

        // 오른쪽 톱니바퀴 확인
        for (int i = gearNum + 1; i < 4; i++) {
            if (gears[i-1][2] != gears[i][6]) {
                rotations[i] = -rotations[i-1];
            } else {
                break;
            }
        }

        // 회전 실행
        for (int i = 0; i < 4; i++) {
            if (rotations[i] != 0) {
                rotateGear(i, rotations[i]);
            }
        }
    }

    static void rotateGear(int gearNum, int direction) {
        if (direction == 1) {
            int temp = gears[gearNum][7];
            for (int i = 7; i > 0; i--) {
                gears[gearNum][i] = gears[gearNum][i-1];
            }
            gears[gearNum][0] = temp;
        } else {
            int temp = gears[gearNum][0];
            for (int i = 0; i < 7; i++) {
                gears[gearNum][i] = gears[gearNum][i+1];
            }
            gears[gearNum][7] = temp;
        }
    }
}
```

**Deque(LinkedList로 구현)자료구조 이용**

```
import java.util.*;
import java.io.*;

public class Main {
    static Deque<Integer>[] gears = new LinkedList[4];
    static int K;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        // 톱니바퀴 상태 입력
        for (int i = 0; i < 4; i++) {
            gears[i] = new LinkedList<>();
            String line = br.readLine();
            for (char c : line.toCharArray()) {
                gears[i].addLast(c - '0');
            }
        }

        // 회전 횟수 입력
        K = Integer.parseInt(br.readLine());

        // 회전 명령 처리
        for (int i = 0; i < K; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            int gearNum = Integer.parseInt(st.nextToken()) - 1;
            int direction = Integer.parseInt(st.nextToken());

            rotate(gearNum, direction);
        }

        // 점수 계산
        int score = 0;
        for (int i = 0; i < 4; i++) {
            if (gears[i].peekFirst() == 1) {
                score += (1 << i);
            }
        }

        System.out.println(score);
    }

    static void rotate(int gearNum, int direction) {
        int[] rotations = new int[4];
        rotations[gearNum] = direction;

        // 왼쪽 톱니바퀴 확인
        for (int i = gearNum - 1; i >= 0; i--) {
            if (getElementAt(gears[i], 2) != getElementAt(gears[i+1], 6)) {
                rotations[i] = -rotations[i+1];
            } else {
                break;
            }
        }

        // 오른쪽 톱니바퀴 확인
        for (int i = gearNum + 1; i < 4; i++) {
            if (getElementAt(gears[i-1], 2) != getElementAt(gears[i], 6)) {
                rotations[i] = -rotations[i-1];
            } else {
                break;
            }
        }

        // 회전 실행
        for (int i = 0; i < 4; i++) {
            if (rotations[i] == 1) {
                gears[i].addFirst(gears[i].removeLast());
            } else if (rotations[i] == -1) {
                gears[i].addLast(gears[i].removeFirst());
            }
        }
    }

    static int getElementAt(Deque<Integer> deque, int index) {
        Iterator<Integer> iterator = deque.iterator();
        for (int i = 0; i < index; i++) {
            iterator.next();
        }
        return iterator.next();
    }
}
```

**Deque(ArrayDeque로 구현)자료구조 이용**

```
import java.util.*;
import java.io.*;

public class Main {
    static ArrayDeque<Integer>[] gears = new ArrayDeque[4];
    static int K;

    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));

        // 톱니바퀴 상태 입력
        for (int i = 0; i < 4; i++) {
            gears[i] = new ArrayDeque<>(8);
            String line = br.readLine();
            for (char c : line.toCharArray()) {
                gears[i].addLast(c - '0');
            }
        }

        // 회전 횟수 입력
        K = Integer.parseInt(br.readLine());

        // 회전 명령 처리
        for (int i = 0; i < K; i++) {
            StringTokenizer st = new StringTokenizer(br.readLine());
            int gearNum = Integer.parseInt(st.nextToken()) - 1;
            int direction = Integer.parseInt(st.nextToken());

            rotate(gearNum, direction);
        }

        // 점수 계산
        int score = 0;
        for (int i = 0; i < 4; i++) {
            if (gears[i].peekFirst() == 1) {
                score += (1 << i);
            }
        }

        System.out.println(score);
    }

    static void rotate(int gearNum, int direction) {
        int[] rotations = new int[4];
        rotations[gearNum] = direction;

        // 왼쪽 톱니바퀴 확인
        for (int i = gearNum - 1; i >= 0; i--) {
            if (getElementAt(gears[i], 2) != getElementAt(gears[i+1], 6)) {
                rotations[i] = -rotations[i+1];
            } else {
                break;
            }
        }

        // 오른쪽 톱니바퀴 확인
        for (int i = gearNum + 1; i < 4; i++) {
            if (getElementAt(gears[i-1], 2) != getElementAt(gears[i], 6)) {
                rotations[i] = -rotations[i-1];
            } else {
                break;
            }
        }

        // 회전 실행
        for (int i = 0; i < 4; i++) {
            if (rotations[i] == 1) {
                gears[i].addFirst(gears[i].removeLast());
            } else if (rotations[i] == -1) {
                gears[i].addLast(gears[i].removeFirst());
            }
        }
    }

    static int getElementAt(ArrayDeque<Integer> deque, int index) {
        return (int) deque.toArray()[index];
    }
}
```
