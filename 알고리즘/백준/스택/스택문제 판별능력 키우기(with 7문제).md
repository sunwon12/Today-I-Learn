### **문제1**   

[https://www.acmicpc.net/problem/2800](https://www.acmicpc.net/problem/2800)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/b02a9a4a-c792-47cb-baf8-0d9dc5186e77)

####  **아이디어**

-   **'('를 스택에 넣은 후 ')' 만날 때마다 괄호클래스 혹은 이차원 배열에 추가해주기**
-   2의 n승(n개의 괄호쌍이 있다, 없다) 의 경우의 수를 어떻게 dfs 재귀함수로 표현할 것인가
    -   dfs 함수 안에 괄호 제거한 버전 괄호 포함한 버전으로 dfs 호출 두 번한다.

**ex)**

```
   private static void dfs(int idx, int len) {
        if (idx == len) {
            print();
            return;
        }

        if (arr[idx] == '(') {
            // 여는 괄호를 만나면 해당 괄호와 짝 괄호를 vist배열에 true처리한다
            visit[idx] = true;
            visit[pair[idx]] = true;
            dfs(idx + 1, len);

            // 해당 괄호 쌍의 visit배열을 다시 false처리한다
            visit[idx] = false;
            visit[pair[idx]] = false;
        }

        dfs(idx + 1, len);
    }
```

-   dfs 식이 유니크한 식을 print하게 보장해주는데 왜 TreeSet을 쓰는지 궁금했다.
    -   중복을 제거해주지 않고 Set 대신 ArrayList를 쓴다면, 문제가 생긴다 아래 예시를 보겠다.   
    -   ((1+2)) 인경우   
        (1+2)  
        (1+2)  
        1+2 가 출력된다.
    -   보다시피 이러한 예외들 때문에 Set을 써야 한다.

### **문제2**   

[https://www.acmicpc.net/problem/13146](https://www.acmicpc.net/problem/13146)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/caa97ae6-38ed-4951-865b-86cf25dd4f27)

#### **아이디어**

-   스택이 비어있을 시 스택에 숫자를 집어넣는다
-   다음 숫자로 넘어간다.
-   스택 맨 위에 있는 숫자와 비교하고  
    스택 맨 위에 있는 숫자보다 크면 스택숫자를 pop시킨다.  
    이를 스택이 빌 때까지 혹은 스택숫자가 더 클 때까지 계속한다.  
    스택 숫자가 더 크다면 비교숫자를 스택에 넣는다. 

### **문제3**   

[https://www.acmicpc.net/problem/13146](https://www.acmicpc.net/problem/13146)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/05a480dd-6713-4a0e-a78c-3eedfbf6aff3)

#### **아이디어**

-   숫자는 StringBuilder.append 한다
-   연산자는 스택에 넣는다
-   ')'을 만난다면 pop을 하여 StringBuilder.append한다.

### **문제4**   

[https://www.acmicpc.net/problem/22343](https://www.acmicpc.net/problem/22343)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/12fc2b33-85b6-4818-8fa1-214f258ecb24)

#### **아이디어**

-   '('를 만나면 Stack 넣는다. ')'를 만나면 count\*2를 한다.  
    단, ')' 바로 전 인덱스에 '('였다면 count\*2가 아닌 count+1를한다.
-   스택이 비어졌다면 realcount += count를 한다.  
    그리고 count를 0으로 초기화한다.
-   이 과정을 한 식의 문자열 끝까지 반복한다.

### **문제5**   

[https://www.acmicpc.net/problem/13146](https://www.acmicpc.net/problem/13146)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/18d0910b-0715-4389-b4bd-f1acc4937194)

솔직히 이 문제가 스택 문제인지 모르고 마주치면 스택문제인지 모를 것 같다. 스택문제인지 알고 봐도 어디서 스택을 써야하는지 모르겠다..

![](https://t1.daumcdn.net/keditor/emoticon/friends1/large/016.gif)

#### **아이디어**

-   가장 큰 수로 똑같아 져야 한다.
    -   수를 입력 받을 때 배열에 추가한다. 그리고 최대값도 업데이트 한다.
-   숫자배열을 왼쪽에서 오른쪽으로 탐색한다.
-   스택이 비어있다면 숫자를 넣어준다.
-   스택 숫자 < 비교숫자이면 
    -   차이만큼 answer+=차이 해주고, 스택 숫자 pop 비교숫자 push
-   스택 숫자 > 비교숫자이면
    -   스택 숫자 pop 비교숫자 push
-   그렇다면 맨 마지막에 스택에 하나의 숫자만 남는다
-   answer += max - 스택에 남은 하나의 숫자

-   **이 스택문제는 그리드처럼 느껴진다.** 
    -   오른쪽으로 갈 수록 숫자들을 지금까지 탐색한 것중 가장 높은 숫자로 통합해가는 거기 때문에 

**배운점**

**배열을 순차적으로 탐색할 때 기준이 업데이트 되거나 답이 업데이트되면 그리도 혹은 Stack을 풀기**

### **문제6**   

[https://www.acmicpc.net/problem/2812](https://www.acmicpc.net/problem/2812)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/f5e4321b-0be2-4a72-9025-a4e9e7f4794f)

**아이디어**

-   첫 번째로 드는 생각 = 완전탐색(DFS)
    -   근데, 시간 초과 날 것 같다 -> 생각 전환
-   먼저 N자리 숫자를 받아 배열을 만든다
-   코드 참고

```
        for(int i = 0; i < str.length() ; i++){
            if(!stack.empty()){
                while(!stack.empty() && K > 0 && stack.peek() < str.charAt(i)){
                    stack.pop();
                    K--;
                }
            }
            
            stack.push(str.charAt(i));
        }
```

-   **위 문제는 풀어본 문제라 스택문제인 것을 알지만 솔직히 처음봤을 때 어떻게 스택문제로 판별해야할지 감이 안온다.**
    -   **괄호 문제 같은 경우는 전에 선택을 저장해놨다가 후에 이것이 나중에 영향을 준다는 사고흐름으로 스택문제인지 판별할 수 있었다. '스카이라인 쉬운거' 문제도 그렇다.**

### **문제7**   

[https://www.acmicpc.net/problem/1863](https://www.acmicpc.net/problem/1863)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/6172a0a2-c2de-4d57-8f0b-90bf76c02f4c)

**아이디어**

-   주어진 입력 값을 차례대로 순회한다.
-   스택이 비어있다면 y좌표를 넣는다.
-   스택y > 비교y라면 (스택y < 비교y 이거나 스택y == 비교y 까지  아래 과정 반복) 
    -   answer++, 스택y pop 과정 스택y < 비교y 이거나 스택y == 비교y 때까지 반복
    -   만약 스택y == 비교 y라면 그냥 넘어가기
    -    만약 스택y < 비교y라면 스택에 넣어주기
