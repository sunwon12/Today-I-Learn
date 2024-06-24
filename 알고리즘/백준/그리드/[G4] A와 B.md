### **문제**         

**레벨: G4  
알고리즘: 그리드**   
**풀이시간: 30분  
힌트 참조 유무: 무**

[https://www.acmicpc.net/problem/12904](https://www.acmicpc.net/problem/12904)

![image](https://github.com/sunwon12/Today-I-Learn/assets/92251131/f1bc2481-5abe-4e18-ba76-7793034af839)

### **1 번째 시도**   

#### **\[로직 설명\]**

T에 룰을 거꾸로 적용해서 하나씩 빼주고 마지막에 비교한다.

#### **\[배운점\]**

String 함수 되짚기

```
import org.w3c.dom.Node;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.ArrayList;
import java.util.Arrays;


public class Main {


    public static void main(String[] args) throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String S = br.readLine();
        String T = br.readLine();
        int N = T.length() - S.length();
        
        for (int i = 0; i < N ; i++) {
            if (T.charAt(T.length()-1) == 'A') {
                T = rule1(T);
            }
            else {
                T = rule2(T);
            }
        }

        if(S.equals(T))
            System.out.println(1);
        else {
            System.out.println(0);
        }

    }

    public static String rule1(String T) {
       return  T.substring(0, T.length()-1);
    }



    public static String rule2(String T) {
        String substring = T.substring(0, T.length() - 1);
        StringBuilder sb = new StringBuilder();
        for(int i = substring.length()-1 ; i >= 0 ; i--) {
            sb.append(substring.charAt(i));
        }
        return sb.toString();
    }
}
```

#### **\[보충해야 할 점\]**

subString을 이용해서 문자열을 빼는 게 아니라 StringBuilder의 deleteCharAt() 함수를 이용할 수 있다.

또 하나, StringBuilder에는 뒤지는 함수 reverse() 가 있다.
