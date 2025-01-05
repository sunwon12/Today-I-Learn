- 옵티마이저가 항상 좋은 실행 계획을 만들어낼 수 있는 것은 아닙니다.
    - 그렇기 때문에 DBMS 서버에서는 이를 보완할 수 있도록 EXPLAIN 명령으로 옵티마이저가 수립한 실행 계획을 확인할 수 있습니다.

# 1. 통계 정보

---

- MySQL 5.7 까지는 테이블과 인덱스에 대한 정보를 가지고 실행 계획을 수립했습니다.
- MySQL 8.0 부터는 **인덱스되지 않은 칼럼**들에 대해서도 데이터 분포도를 수집해서 저장하는 히스토그램 정보가 도입됐습니다.

## 1.1 테이블 및 인덱스 통계 정보

---

- **MySQL 서버의 통계 정보** MySQL 5.5 버전 까지는 각 테이블의 통계 정보가 메모리에서만 관리되어서 서버 재시작시 모두 사라졌습니다.
- 이후의 버전에서는 각 테이블의 통계 정보를 `mysql` 데이터베이스의 `innodb_index_stats` 테이블과 `innodb_index_stats` 테이블로 관리할 수 있게되어 **재시작** 되어도 통계 정보가 **유지됩니**다.
    - 특정 테이블의 통계 정보를 영구적으로 관리하고 싶지 않을 경우 테이블 생성 시 `STATS_PERSISTENT`를 0으로 설정하면 됩니다.`STATS_PERSISTENT`를 설정하지 않으면 `innodb_stats_persistent` 시스템 변수의 값에 따라 결정합니다. (ON이면 영구 저장, OFF면 영구 저장 X)

```sql

-- board_statistic 테이블의 인덱스 통계 정보
SELECT *
FROM mysql.innodb_index_stats
WHERE database_name = 'bakery'
  AND table_name = 'board_statistic';
  
```

![image](https://github.com/user-attachments/assets/76e5965a-6447-4d15-92be-5973feb9a86a)

```java
  
-- board_statistic 테이블의 통계 정보
 SELECT *
 FROM mysql.innodb_table_stats
 WHERE database_name='bakery'
   AND TABLE_NAME='board_statistic';
```

![image](https://github.com/user-attachments/assets/dc94b915-d5b8-466c-83cc-af133507bb89)

아래와 같은 이벤트가 발생하면 자동으로 통계 정보가 갱신됩니다.

- 테이블이 새로 오픈되는 경우
- 테이블의 레코드가 대량으로 변경되는 경우
- ANALYZE TABLE 명령이 실행되는 경우
- SHOW TABLE STATUS 명령이나 SHOW INDEX FROM 명령이 실행되는 경우
- InnoDB 모니터가 활성화되는 경우
- innodb_status_on_metadata 시스템 설정이 ON인 상태에서 SHOW TABLE STATUS 명령이 실행되는 경우

## 1.2 **히스토그램**

---

- MYSQl 5.7 버전까지의 통계 정보는 단순히 인덱스된 칼럼의 유니크한 값의 개수 정도만 가지고 있었는데, 이는 옵티마이저가 최적의 실행 계획을 수립하기에는 많이 부족했습니다.
- MySQL 8.0부터 도입된 히스토그램은 `데이터 분포`를 나타내는 통계 정보로, 쿼리 최적화에 중요한 역할을 합니다.
- 히스토그램 정보는 컬럼 단위로 관리되는데, 이는 자동으로 수집되지 않고 `ANALYZE TABLE … UPDATE HISTOGRAM` 명령을 실행해 수동으로 수집됩니다.
- 이 히스토그램 정보를 조회하려면 `column_statistics` 테이블을 `SELECT`하면 됩니다.
- 히스토그램은 싱글톤 히스토그램, 높이 균형 히스토그램 두 가지가 지원합니다.
    - **SingleTon(싱글톤 히스토그램)**: 칼럼값 개별로 레코드 건수를 관리하는 히스토그램
    - **Equi-Height(높이 균형 히스토그램):** 칼럼값의 범위를 균등한 개수로 구분해서 관리하는 히스토그램

### 1.2.1 히스토그램이 없을 때와 히스토그램이 있을 때 예측치 비교

---

**조건**: 테이블 레코드가 1000건이고 유니크한 값이 100개인 데이터셋

```sql
## 1. 히스토그램이 없는 상태에서 실행 계획 확인

EXPLAIN
SELECT *
FROM employees
WHERE first_name = 'Zita'
  AND birth_date BETWEEN '1950-01-01' AND '1960-01-01';

```

**실행 계획 결과**

| id | select_type | table | type | key | rows | filtered |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | employees | ref | ix_firstname | 224 | 11.11 |

```sql
## 히스토그램 생성 후 실행 계획 확인

ANALYZE TABLE employees
UPDATE HISTOGRAM ON first_name, birth_date;

EXPLAIN
SELECT *
FROM employees
WHERE first_name = 'Zita'
  AND birth_date BETWEEN '1950-01-01' AND '1960-01-01';

```

### 실행 계획 결과

| id | select_type | table | type | key | rows | filtered |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | employees | ref | ix_firstname | 224 | 60.82 |
|  |  |  |  |  |  |  |

```sql
## 실제 데이터 분포 확인

SELECT
  SUM(CASE WHEN birth_date BETWEEN '1950-01-01' AND '1960-01-01' THEN 1 ELSE 0 END) / COUNT(*) AS ratio
FROM employees
WHERE first_name = 'Zita';

```

### 실제 결과

| ratio |
| --- |
| 0.6384 |
- 히스토그램 생성 이전:
    - 옵티마이저는 조건 필터링(`filtered`) 값을 `11.11%`로 잘못 예측.
    - 균등하게 분포돼 있을 것으로 예측하기 때문 100/1000
- 히스토그램 생성 이후:
    - 옵티마이저가 조건 필터링 값을 `60.82%`로 근접하게 예측.
- 실제 데이터 결과:
    - 조건 필터링의 실제 값은 약 `63.84%`.
    - 히스토그램이 옵티마이저의 예측 정확도를 높이는 데 기여.

### 1.2.2 예측치가 중요한 이유

---

11.11퍼센트로 예측치를 잘못 계산했다면 조인의 순서가 바뀔 수도 있는 것이고 인덱스 풀 스캔 해야 할 것을 인덱스 레인지 스캔으로 하여 쿼리 비용이 더 상승할 수 있습니다.

## 1.3 코스트 모델

---

전체 쿼리의 비용을 계산하는데 필요한 단위 작업들의 비용을 코스트 모델이라고 합니다.

MySQL의 코스트 모델은 다음 2개 테이블에 저장되어 있는 설정값을 사용합니다.

- `server_cost`: 인덱스를 찾고 레코드를 비교하고 임시 테이블 처리에 대한 비용 관리
- `engine_cost`: 레코드를 가진 데이터 페이지를 가져오는데 필요한 비용 관리

두 테이블은 아래 5개의 컬럼을 공통으로 가지고있다.

- `cost_name`: 코스트 모델의 각 단위 작업
- `default_value`: 각 단위 작업의 비용(기본값, MySQL 서버 소스 코드에 설정된 값)
- `cost_value`: DBMS 관리자가 설정한 값(NULL이면 `default_value` 값 사용)
- `last_updated`: 단위 작업의 비용이 변경된 시점
- `comment`: 비용에 대한 추가 설명

`engine_cost` 테이블은 아래 2개의 컬럼을 추가로 가지고 있습니다.

- `engine_name`: 비용이 적용된 스토리지 엔진
- `device_type`: 디스크 타입

단위 작업의 종류는 아래와 같이 8개가 있다.

| 테이블 이름 | cost_name | default_value | 설명 | 특징 |
| --- | --- | --- | --- | --- |
| engine_cost | io_block_read_cost | 1 | 디스크 데이터 페이지 읽기 |  |
| engine_cost | memory_block_read_cost | 0.25 | 메모리 데이터 페이지 읽기 |  |
| server_cost | disk_temptable_cost | 20 | 디스크 임시 테이블 생성 | 해당 비용이 높으면 `디스크`에 `임시 테이블`을 만들지 않는 방향으로 선택할 가능성이 높음 |
| server_cost | disk_temptable_row_cost | 0.5 | 디스크 임시 테이블의 레코드 읽기 | 해당 비용이 높으면 `디스크`에 `임시 테이`블을 만들지 않는 방향으로 선택할 가능성이 높음 |
| server_cost | key_compare_cost | 0.05 | 인덱스 키 비교 | 해당 비용이 높으면 `정렬`을 수행하지 않는 방향으로 선택할 가능성이 높음 |
| server_cost | memory_temptable_create_cost | 1 | 메모리 임시 테이블 생성 | 해당 비용이 높으면 `메모리`에 `임시 테이블`을 만들지 않는 방향으로 선택할 가능성이 높음 |
| server_cost | memory_temptable_row_cost | 0.1 | 메모리 임시 테이블의 레코드 읽기 | 해당 비용이 높으면 `메모리`에 `임시 테이블`을 만들지 않는 방향으로 선택할 가능성이 높음 |
| server_cost | row_evaluate_cost | 0.1 | 레코드 비교 | 해당 비용이 높으면 풀 스캔 비용이 높다는 것이어서 `인덱스 레인지 스캔`을 선택할 가능성이 높음 |

코스트 모델은 각 단위 작업에 설정되는 비용이 커지면 어떤 실행 계획의 비용이 변하는지 파악하는 것이 중요하다.

웬만하면 위 테이블들의 `default_value`를 바꾸지 말자

### 1.3.1 코스트 확인하는 방법

---

```sql
EXPLAIN FORMAT = JSON
SELECT r.id, r.content, r.rate, r.badge_taste, r.badge_brix, r.badge_texture,
       COUNT(rl.id) as like_count
FROM review r
    LEFT JOIN review_like rl ON r.id = rl.review_id
WHERE r.board_id = 1 
  AND r.is_deleted = false
GROUP BY r.id;
```

```sql
{
    "query_block": {
        "select_id": 1,
        "cost": 0.13062319,
        "nested_loop": [
            {
                "table": {
                    "table_name": "r",
                    "access_type": "ref",
                    "possible_keys": [
                        "idx_board_id"
                    ],
                    "key": "idx_board_id",
                    "key_length": "9",
                    "used_key_parts": [
                        "board_id"
                    ],
                    "ref": [
                        "const"
                    ],
                    "loops": 1,
                    "rows": 46,
                    "cost": 0.08502736,
                    "filtered": 100,
                    "attached_condition": "r.board_id <=> 1 and r.is_deleted = 0"
                }
            },
            {
                "table": {
                    "table_name": "rl",
                    "access_type": "ref",
                    "possible_keys": [
                        "fk_review_review_like"
                    ],
                    "key": "fk_review_review_like",
                    "key_length": "8",
                    "used_key_parts": [
                        "review_id"
                    ],
                    "ref": [
                        "bakery.r.id"
                    ],
                    "loops": 46,
                    "rows": 1,
                    "cost": 0.04559583,
                    "filtered": 100,
                    "using_index": true
                }
            }
        ]
    }
}
```

# 2. 실행 계획 확인

---

MySQL의 실행 계획은 `DESC` 또는 `EXPLAIN` 명령으로 확인할 수 있습니다.

실행 계획의 포맷은 아래와 같이 테이블, 트리, JSON 3가지 중 하나를 선택할 수 있다.

- `EXPLAIN [FORMAT=TREE or JSON]` (테이블이 기본값)
- 8.0.18부터 `EXPLAIN ANALYZE` 추가
    - 단계별 소요된 시간 정보 확인 가능
    - 항상 TREE 포맷으로 보여줌

## 2.1 쿼리의 실행 시간 확인

---

**실행 쿼리 (TREE 포맷)**

```sql

EXPLAIN ANALYZE
SELECT e.emp_no, avg(s.salary)
FROM employees e
INNER JOIN salaries s ON s.emp_no = e.emp_no
                     AND s.salary > 50000
                     AND s.from_date <= '1990-01-01'
                     AND s.to_date > '1990-01-01'
WHERE e.first_name = 'Matt'
GROUP BY e.hire_date \G

```

**출력 결과 (TREE 포맷)**

![image](https://github.com/user-attachments/assets/8d311e82-1029-47a1-9b65-15b054590670)


### **TREE 포맷 해석 규칙**

- **들여쓰기가 같은 레벨**에서는 **상단 라인**이 먼저 실행됩니다.
- **들여쓰기가 다른 레벨**에서는 **가장 안쪽 라인**부터 실행됩니다.

---

### **실제 실행 순서**

1. **D)** `employees` 테이블의 `ix_firstname` 인덱스를 이용해 `first_name='Matt'` 조건에 일치하는 레코드를 찾음.
2. **F)** `salaries` 테이블의 `PRIMARY` 키를 사용해 `emp_no`를 기준으로 일치하는 레코드 검색.
3. **E)** `s.salary > 50000` 및 날짜 조건을 필터링.
4. **C)** `Nested loop inner join`으로 두 테이블을 조인.

1. **B)** 임시 테이블에 결과를 저장하고 `GROUP BY e.hire_date`를 적용. 

1. `← 임시 테이블은 어쩔 때 생기나`

1. **A)** 임시 테이블의 결과를 반환.

---

### 📈 **결과 해석**

F단계를 예시로 해석해보겠습니다.

### **1. actual time**

- `actual time=0.007..0.009`
    - **0.007ms**: 첫 번째 레코드를 가져오는 데 걸린 시간.
    - **0.009ms**: 마지막 레코드를 가져오는 데 걸린 시간.

### **2. rows**

- `rows=10`: 인덱스를 사용하여 읽은 레코드의 예상 수.

### **3. loops**

- `loops=233`: 해당 작업이 **233번 반복**되었음을 의미.

---

### 📊 **EXPLAIN ANALYZE와 EXPLAIN 비교**

| **특징** | EXPLAIN | EXPLAIN ANALYZE |
| --- | --- | --- |
| **쿼리 실행 여부** | 쿼리를 실행하지 않음 | **실제로 쿼리를 실행** |
| **소요 시간 제공** | 제공하지 않음 | **단계별 소요 시간 제공** |
| **출력 포맷** | TABLE, JSON, TREE 포맷 지원 | **항상 TREE 포맷 사용** |
| **주요 사용 목적** | 실행 계획 사전 확인 | **실행 성능 분석** |

## 2.2 실행 계획 분석

---

### **📊 EXPLAIN 예제**

```sql
EXPLAIN
SELECT *
FROM employees e
INNER JOIN salaries s
ON s.emp_no = e.emp_no
WHERE first_name = 'ABC';

```

### 실행 계획 결과 (TABLE 포맷)

| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | e | NULL | ref | PRIMARY,ix_firstname | ix_firstname | 58 | const | 1 | 100 | NULL |
| 1 | SIMPLE | s | NULL | ref | PRIMARY | PRIMARY | 4 | employees.e.emp_no | 10 | 100 | NULL |

---

### 📌 `EXPLAIN` 결과 컬럼별 간단 설명

**실행 순서는 위에서 아래**로 순서대로 표시됩니다.(UNION이나 상관 서브쿼리와 같은 경우 순서대로 표시되지 않을 수도 있습니다.)

- **id**: 각 SELECT 단위를 식별하는 번호. 여러 개의 `SELECT`가 포함되면 1부터 번호가 할당됩니다.
- **select_type**: 쿼리의 유형을 설명합니다. (`SIMPLE`, `PRIMARY`, `SUBQUERY` 등)
- **table**: 조회하는 테이블 이름.
- **type**: 테이블 접근 방식 (`ref`, `index`, `ALL` 등)
- **possible_keys**: 옵티마이저가 고려한 인덱스 목록.
- **key**: 최종적으로 선택된 인덱스.
- **key_len**: 사용된 인덱스 키 길이(바이트 단위).
- **ref**: 인덱스와 비교되는 값.
- **rows**: 예상 레코드 수.
- **filtered**: 조건을 만족할 것으로 예측되는 비율.
- **Extra**: 추가적인 정보 (ex. `Using index`, `Using where`).

## 2.2.1 `id` 칼럼

---

### **`id` 칼럼 설명:**

- `id`는 각 `SELECT` 쿼리를 구분하는 식별자.
- 실행 순서는 **항상 `id` 순서대로 진행되는 것은 아님.**
- **실행 순서 확인을 위해** `EXPLAIN FORMAT=TREE` 또는 `EXPLAIN ANALYZE`를 사용.
- **단일 쿼리**: 모든 `SELECT`가 동일한 `id`를 가짐.
- **서브쿼리 사용**: 각 `SELECT`마다 고유한 `id` 할당.

### **예제: 하위 SELECT 사용**

```sql
SELECT ...
FROM (SELECT * FROM tb_test1) tb1, tb_test2 tb2
WHERE tb1.id = tb2.id;

```

### 결과:

- `tb_test1` 서브쿼리는 `id=2`
- 외부 쿼리는 `id=1`

---

### **다중 테이블 조회 시 `id` 값 동일한 경우**

```sql
EXPLAIN
SELECT e.emp_no, e.first_name, s.from_date, s.salary
FROM employees e, salaries s
WHERE e.emp_no = s.emp_no LIMIT 10;

```

| id | select_type | table | type | key | ref | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | e | index | ix_firstname | NULL | 300252 | Using index |
| 1 | SIMPLE | s | ref | PRIMARY | employees.e.emp_no | 10 | NULL |
- `id=1`로 두 테이블이 동일한 쿼리 블록에서 처리됨.

---

### **서브쿼리 사용 시 `id` 값이 다르게 나오는 경우**

```sql
EXPLAIN
SELECT (
    SELECT COUNT(*) FROM employees
    + SELECT COUNT(*) FROM departments
) AS total;

```

| id | select_type | table | type | key | ref | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | PRIMARY | NULL | NULL | NULL | NULL | NULL | No tables used |
| 3 | SUBQUERY | departments | index | ux_deptname | NULL | 9 | Using index |
| 2 | SUBQUERY | employees | index | ix_hiredate | NULL | 300252 | Using index |
- 각 `SELECT` 구문에 고유한 `id`가 부여됨.
- 앞서 말씀드렸던 것처럼 위아래 순서와 상관없이 id 값이 순서를 의미합니다.

## 2.2.2 `select_type` 칼럼

---

**select_type 칼럼은 MySQL 실행 계획에서 각 `SELECT` 문이 어떤 유형의 쿼리인지 설명합니다.**

### 2.2.2.1 SIMPLE

---

 

- 서브쿼리를 사용하지 않는 단순한 `SELECT` 문입니다.
- 특징:
    - 하나의 `SELECT`만 포함된 경우.
    - 조인을 포함하더라도 서브쿼리가 없으면 SIMPLE로 표시됩니다.
    - 일반적으로 가장 바깥의 SELECT 문에서 사용됩니다.

### 2.2.2.2 PRIMARY

---

- UNION`이나 서브쿼리가 존재할 때 가장 외곽(루트)에 위치한 SELECT 문입니다.
- 특징:
    - 여러 **`SELECT`**중 가장 외곽의 쿼리에 사용됩니다.
    - 서브쿼리 없이 단순한 경우 `SIMPLE`과 동일하지만, 서브쿼리가 있다면 `PRIMARY`로 변경됩니다.
    

### 2.2.2.3 UNION

---

- `UNION`이나 `UNION ALL`로 결합된 쿼리 중 두 번째 이후의 `SELECT` 문에 사용됩니다.
- 특징:
    - 여러 `SELECT`중 가장 외곽의 쿼리에 사용됩니다.
    - 서브쿼리 없이 단순한 경우 `SIMPLE`과 동일하지만, 서브쿼리가 있다면 `PRIMARY`로 변경됩니다.
    

### **예제코드**

```sql
EXPLAIN
SELECT * FROM (
    (SELECT emp_no FROM employees e1 LIMIT 10)
    UNION ALL
    (SELECT emp_no FROM employees e2 LIMIT 10)
    UNION ALL
    (SELECT emp_no FROM employees e3 LIMIT 10)
) tb;
```

### 실행 계획

| id | select_type | table | type | key | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | PRIMARY | derived2 | ALL | NULL | NULL | NULL |
| 2 | DERIVED | e1 | index | ix_hiredate | 30 | Using index |
| 3 | UNION | e2 | index | ix_hiredate | 300252 | Using index |
| 4 | UNION | e3 | index | ix_hiredate | 300252 | Using index |
- **`PRIMARY`**: 메인 쿼리를 의미.
- **`DERIVED`**: 첫 번째 `SELECT` 문은 `DERIVED`로 처리되며 임시 테이블 생성.
- **`UNION`**: 두 번째, 세 번째 `SELECT`는 `UNION`으로 표시.

### 2.2.2.4 DEPENDENT UNION

---

- **설명:** `UNION`의 결과가 외부 쿼리에 영향을 받는 경우 사용됩니다.
- **특징:** 외부 쿼리에 의존하는 `UNION`이 있을 때, 서브쿼리가 실행되는 방식입니다.

### 예제:

```sql
EXPLAIN
SELECT *
FROM employees e1. ## <--- 외부 쿼리
WHERE e1.emp_no IN (
    SELECT e2.emp_no FROM employees e2 WHERE e2.first_name = 'Matt'
    UNION
    SELECT e3.emp_no FROM employees e3 WHERE e3.last_name = 'Matt'
);

```

### 실행 계획

| id | select_type | table | type | key | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | PRIMARY | e1 | ALL | NULL | 300252 | Using where |
| 2 | DEPENDENT SUBQUERY | e2 | eq_ref | PRIMARY | 1 | Using where |
| 3 | DEPENDENT UNION | e3 | eq_ref | PRIMARY | 1 | Using where |
| 4 | UNION RESULT | NULL | NULL | NULL | NULL | Using temporary |
- **`PRIMARY`**: 외부 쿼리.
- **`DEPENDENT SUBQUERY`**, **`DEPENDENT UNION`**: 외부 쿼리에 의존하는 서브쿼리.
- **`UNION RESULT`**: 최종 `UNION` 결과를 임시 테이블에 담고 출력.

### 2.2.2.5 UNION RESULT

---

- **설명:** `UNION`의 결과를 담는 임시 테이블을 의미합니다.
- **특징:** `UNION`이나 `UNION DISTINCT` 사용 시 최종 결과를 저장하기 위해 사용됩니다.

### 예제:

```sql
EXPLAIN
SELECT emp_no FROM salaries WHERE salary > 100000
UNION DISTINCT
SELECT emp_no FROM dept_emp WHERE from_date > '2001-01-01';

```

### 실행 계획

| id | select_type | table | type | key | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | PRIMARY | salaries | range | ix_salary | 191348 | Using where; Using index |
| 2 | UNION | dept_emp | range | ix_fromdate | 5325 | Using where; Using index |
| 3 | UNION RESULT | NULL | NULL | NULL | NULL | Using temporary |

### 2.2.2.6 SUBQUERY

---

- **설명:** 서브쿼리에서 사용되는 `SELECT` 문을 나타냅니다.
- **특징:**
    - `FROM` 절 외에서 사용되는 서브쿼리를 설명합니다.
    - `FROM` 절에 서브쿼리가 있다면 `DERIVED` 로 표시됩니다.

### 예제:

```sql
EXPLAIN
SELECT e.first_name,
    (SELECT COUNT(*)
     FROM dept_emp de, dept_manager dm
     WHERE dm.dept_no = de.dept_no) AS cnt
FROM employees e
WHERE e.emp_no = 10001;

```

### 실행 계획

| id | select_type | table | type | key | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | PRIMARY | e | const | PRIMARY | 1 | NULL |
| 2 | SUBQUERY | dm | index | PRIMARY | 24 | Using index |
| 3 | SUBQUERY | de | ref | PRIMARY | 41392 | Using index |

### 2.2.2.7 DEPENDENT SUBQUERY

---

### 예제:

```sql
mysql> EXPLAIN
SELECT e.first_name,
       (SELECT COUNT(*)
        FROM dept_emp de, dept_manager dm
        WHERE dm.dept_no=de.dept_no AND de.emp_no=e.emp_no) AS cnt
FROM employees e
WHERE e.first_name='Matt';

```

### **실행 계획:**

| id | select_type | table | type | key | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | PRIMARY | e | ref | ix_firstname | 233 | Using index |
| 2 | DEPENDENT SUBQUERY | de | ref | ix_empno_fromdate | 2 | Using index |
| 2 | DEPENDENT SUBQUERY | dm | ref | PRIMARY | 1 | Using index |
- 서브쿼리가 바깥쪽 쿼리에 정의된 `e.first_name`에 의존하기 때문에 `DEPENDENT` 키워드가 붙음.
- 외부 쿼리가 먼저 수행된 후, 내부 서브쿼리가 실행되어야 함.
- `DEPENDENT` 키워드가 없는 쿼리보다 성능 저하 발생 가능.

### 2.2.2.8 DERIVED

---

### 예제:

```sql
mysql> EXPLAIN
SELECT *
FROM (SELECT de.emp_no FROM dept_emp de GROUP BY de.emp_no) tb,
     employees e
WHERE e.emp_no=tb.emp_no;

```

### 실행 계획:

| id | select_type | table | type | key | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | PRIMARY | <derived2> | ALL | NULL | 331143 | NULL |
| 1 | PRIMARY | e | eq_ref | PRIMARY | 1 | NULL |
| 2 | DERIVED | de | index | ix_empno_fromdate | 331143 | Using index |
- 이전 MySQL 5.5 버전까지 `FROM` 절에서 사용된 서브쿼리는 항상 `select_type`이 `DERIVED`로 처리되었음.
- `DERIVED`는 `FROM`절 서브쿼리를 위해 메모리나 디스크에 `임시 테이블`을 생성하는 방식.
- MySQL 8.0 이후부터 옵티마이저가 FROM절 서브쿼리를 조인으로 최적화.

### 2.2.2.9 DEPENDENT DERIVED

---

MySQL 8.0부터 `LATERAL JOIN`이 추가되면서 외부 칼럼을 참조할 수 있게 되었다.

### 예제:

```sql
mysql> EXPLAIN
SELECT *
FROM employees e
LEFT JOIN LATERAL (
    SELECT *
    FROM salaries s
    WHERE s.emp_no=e.emp_no
    ORDER BY s.from_date DESC LIMIT 2
) AS s2 ON s2.emp_no=e.emp_no;

```

### 실행 계획:

| id | select_type | table | type | key | Extra |
| --- | --- | --- | --- | --- | --- |
| 1 | PRIMARY | e | ALL | NULL | Rematerialize(<derived2>) |
| 1 | PRIMARY | <derived2> | ref | <auto_key> | NULL |
| 2 | DEPENDENT DERIVED | s | ref | PRIMARY | Using filesort |

### 2.2.2.10 UNCACHEABLE SUBQUERY

---

### 예제:

```sql
mysql> EXPLAIN
SELECT *
FROM employees e
WHERE e.emp_no = (
    SELECT @status FROM dept_emp de WHERE de.dept_no='d005'
);

```

### 실행 계획:

| id | select_type | table | type | key | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | PRIMARY | e | ALL | NULL | 300252 | Using where |
| 2 | UNCACHEABLE SUBQUERY | de | ref | PRIMARY | 165571 | Using index |
- 조건이 똑같은 SUBQUERY와 DEPENDENT SUBQUERY는 이전의 실행 결과를 그대로 사용할 수 있게 서브쿼리의 결과를 내부적인 캐시 공간에 담아둡니다.(캐시 공간 ≠ 임시테이블)
- 서브쿼리가 실행될 때마다 매번 결과를 새로 계산해야 하는 경우 `UNCACHEABLE SUBQUERY`로 표시됨.

### 2.2.2.11 UNCACHEABLE UNION

---

- `UNCACHEABLE UNION`은 `UNCACHEABLE SUBQUERY`와 비슷하지만, `UNION ALL` 쿼리에서 적용됨.

### 2.2.2.12 MATERIALIZED

---

### 예제:

```sql
mysql> EXPLAIN
SELECT *
FROM employees e
WHERE e.emp_no IN (SELECT emp_no FROM salaries WHERE salary BETWEEN 100 AND 1000);

```

### 실행 계획:

| id | select_type | table | type | key | rows | Extra |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | <subquery2> | ALL | NULL | NULL | NULL |
| 1 | SIMPLE | e | eq_ref | PRIMARY | 1 | NULL |
| 2 | MATERIALIZED | salaries | range | ix_salary | 1 | Using where; Using index |
- MySQL 5.6 이후 `select_type`으로 `MATERIALIZED`가 도입됨.
- 서브쿼리의 내용을 임시 테이블로 구체화(`MATERIALIZED`)한 후, 임시테이블과 emlployees 테이블을 조인하는 형태로 최적화되어 처리.
- 주로 `FROM` 절이나 `IN (subquery)` 형태에서 사용되는 서브쿼리를 최적화하는데 사용.
    
    

## 2.2.3 `table` 칼럼

---

### 실행 계획:

| id | select_type | table |
| --- | --- | --- |
| 1 | PRIMARY | <derived2> |
| 1 | PRIMARY | e |
| 2 | DERIVED | de |
- <derived2>의 의미
    - SELECT 쿼리의 id 값이 2인 실행 계획으로부터 만들어진 파생 테이블(임시 테이블)을 가르킴.
- table 컬럼을 이용하여 순서 파악
    - 첫 번째 라인의 테이블이 <derived2>으로 되어 있으니 임시 테이블이 준비되어있어야 함을 알수 있음.
    - 세 번째 라인 라인의 테이블을 보면 de 테이블을 읽어서 파생 테이블을 만드는 것을 알 수 있음.
    - 세 번째 라인 분석이 끝났으므로 다시 실행 계획의 첫 번째 라인으로 돌아감.
    - 첫 번째 라인과 두 번째 라인은 같은 id 값을 가지고 있는 것으로 봐서 2개 테이블(derived2, e)이 조인되는 쿼리라는 사실 알 수 있음.
        - derived2가 e보다 윗 라인에 있기 때문에 derived2가 드라이빙 테이블이 되고 e가 드리븐 테이블이 되는 것을 알 수 있음
        - 즉, derived2 테이블을 먼저 읽어서 e 테이블로 조인을 실행.
- select_type 칼럼의 값 `MATERIALIZED` 계획에서는 <subqueryN>과 같은 값이 표시. 이는 서브쿼리의 결과를 구체화해서 임시 테이블로 만들었다는 의미. <derived N>과 똑같이 해석해도 무방.

## 2.2.4 `type` 칼럼 (접근 방식)

---

- 어떤 방식(인덱스를 사용했는지, 풀 테이블 스캔인지 …)으로 읽어왔는지를 나타냅니다.

### **`type`의 종류 (효율 순서)**

1. **system**: 테이블에 레코드가 1건만 있는 경우. **`(가장 효율)`**
    1. InnoDB에서는 안 쓰임
2. **const**: PK나 UNIQUE KEY로 정확히 1건 조회.
    1. 다중 유니크 인덱스에서 한 개의 컬럼으로만 조건절 사용할 경우 값이 하나라는 것을 보장할 수 없으므로 `ref` 
3. **eq_ref**: 조인 시 PK나 유니크 키를 사용한 경우.
4. **ref**: 비유니크 인덱스를 사용한 검색.
5. **range**: 인덱스 범위를 사용한 검색 (`BETWEEN`, `<`, `>`)
6. **index**: 인덱스 전체 스캔 (성능 낮음).
7. **ALL**: 풀 테이블 스캔 **`(가장 비효율적).`**

이외의 

- **fulltext**: 전문 검색 인덱스 사용
- **ref_or_null**: ref 방식 또는 null 비교 접근 방법
- **unique_subquery:** where 조건 절에서 사용될 수 있는 ****In(서브쿼리) 형태의 쿼리 접근 방법
- **index_subquery:** unique_subquery의 In(서브쿼리) 중복된 값을 제거할 때
- **index_merge**: OR 연산자로 연결된 쿼리, 각 두 개의 조건 조회 후 병합

## 2.2.5 `possible_keys` 칼럼

---

**사용될 법**했던 인덱스의 목록입니다. 실제로 그 인덱스를 사용했다는 게 아닙니다. 특별한 경우를 제외하고는 그냥 무시해도 됩니다. 

## 2.2.6 `key` 칼럼

---

최종으로 선택되어 사용되는 인덱스를 의미합니다. 쿼리를 튜닝할 때는 key 칼럼에 의도했던 인덱스가 표시되는지 확인하는 것이 중요합니다. type 칼럼이 index_merge인 경우만 제외하면 1개 인덱스만 사용됩니다.

type 칼럼이 ALL(풀 테이블 스캔)일 때 인덱스를 전혀 사용하지 못함으로 key 칼럼은 NULL로 표시됩니다.

## 2.2.7 `key_len` 칼럼

---

실무에서는 다중 컬럼 인덱스를 사용할 때가 많습니다. key_len의 값을 보고 하나의 인덱스만 사용했는지, 여러 개의 인덱스만 사용했는지 유추해볼 수 있습니다. 

### 📊 **예제 쿼리 및 실행 계획:**

```sql
EXPLAIN
SELECT *
FROM dept_emp
WHERE dept_no = 'd005';
```

**실행 계획 결과 (테이블 포맷):**

| id | select_type | table | type | key | key_len |
| --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | dept_emp | ref | PRIMARY | 16 |

**분석:**

- **`key_len: 16`**
    - `dept_no` 칼럼의 타입이 `CHAR(4)`이므로 `utf8mb4` 인코딩에서는 1문자가 **최대 4바이트**를 사용합니다.
    - `CHAR(4)` → `4 * 4 = 16바이트`
    - 따라서 `key_len` 값이 `16`으로 계산되었습니다.
- **`type: ref`**
    - 다중 컬림 인덱스를 다 사용하지 않았으니 const가 아닌 `참조 검색(ref)`이 발생했음을 의미합니다.

**+) 추가적으로 인덱스 데이터 바이트 수보다 추가 정보로 인해 약간은 더 높게 나올 수 있습니다.**

## 2.2.8 `ref` 칼럼

---

type 컬럼이 ref나 eq_ref일 때 표시되는 컬럼입니다. 조인 대상 칼럼의 이름이 그대로 표시됩니다.

## 2.2.9 `row` 칼럼

---

실행 계획의 rows 칼럼값은 쿼리를 처리하기 위해 읽어야 하는 레코드 건수 **예측값**.

## 2.2.10 `filtered` 칼럼

---

- **`filtered` 값은 인덱스를 사용한 이후, 추가로 필터링되는 비율**을 의미합니다.
- MySQL 서버 옵티마이저는 레코드 건수뿐만 아니라 다른 요소들도 충분히 감안해서 실행 계획을 수립하겠지만, 조인의 횟수를 줄이고 그 과정에 읽어온 데이터를 저장해둘 메모리 사용량을 낮추기 위해 대상 건수가 적은 테이블을 선행 테이블로 선택할 가능성이 높습니다.
- MySQL 8.0에서는 `filtered` 칼럼의 값을 더 정확히 예측할 수 있도록 히스토그램 기능이 도입되었습니다.

```sql
EXPLAIN
SELECT *
FROM employees e,
     salaries s
WHERE e.first_name = 'Matt'
  AND e.hire_date BETWEEN '1990-01-01' AND '1991-01-01'
  AND s.emp_no = e.emp_no
  AND s.from_date BETWEEN '1990-01-01' AND '1991-01-01'
  AND s.salary BETWEEN 50000 AND 60000;
```

### 실행 계획

| id | select_type | table | type | key | rows | filtered |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | e | ref | ix_firstname | 233 | 16.03 |
| 1 | SIMPLE | s | ref | PRIMARY | 10 | 0.48 |

**설명:**

- employees 테이블에서 `e.first_name='Matt'` 인덱스 조건에 일치하는 레코드는 233건이고 `AND e.hire_date BETWEEN '1990-01-01' AND '1991-01-01'` 조건까지 만족하는 레코드는 37건(233 * 0.1603)입니다.

---

### 변경된 쿼리 (힌트 사용)

```sql
EXPLAIN
SELECT /*+ JOIN_ORDER(s, e) */
FROM employees e,
     salaries s
WHERE e.first_name = 'Matt'
  AND e.hire_date BETWEEN '1990-01-01' AND '1991-01-01'
  AND s.emp_no = e.emp_no
  AND s.from_date BETWEEN '1990-01-01' AND '1991-01-01'
  AND s.salary BETWEEN 50000 AND 60000;

```

### 실행 계획

| id | select_type | table | type | key | rows | filtered |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | s | range | ix_salary | 3314 | 11.11 |
| 1 | SIMPLE | e | eq_ref | PRIMARY | 1 | 5.00 |

**설명:**

- `salaries` 테이블을 드라이빙 테이블로 선택하여 먼저 검색합니다.
- `salaries` 테이블에서 인덱스를 활용하여 3314개의 레코드를 찾고, 그 중 11.11%인 368건으로 필터링됩니다.

**요약**:

- `rows * filtered/100 = 실제 남은 레코드 건수`
- 1 번째 방법으로 조인 시 드리븐 테이블에 37 번만 접근할 수 있지만, 2 번째 방법으로 조인 시 368건 조인해야 합니다. 그래서 옵티마이저는 filtered를 확인하고 1 번째 순서로 조인할 것입니다.

## 2.2.5 `Extra` 칼럼

---

`Extra`는 추가적인 실행 계획 정보를 제공합니다. 세미콜론(;)으로 구분하며 30여 가지가 있습니다.

- **Using index**: **커버링 인덱스** 사용.
- **Using where**: 인덱스를 사용하지 않고, 데이터 파일을 읽은 후 필터링을 수행한다는 의미입니다.
    
      → **성능 저하 가능성 있음.**
    
- **Using temporary**: 임시 테이블을 사용함.  → **디스크 I/O 발생 가능성.**
    - 임시 테이블은 `GROUP BY`, `DISTINCT`, `UNION`과 같은 연산이 포함될 때 생성
- **Using filesort**: 정렬이 필요함 (`ORDER BY`). 소트 버퍼 사용  → **성능 저하 위험**
    - `ORDER BY` 칼럼에 적절한 인덱스를 추가해서 성능 개선해야 함
- **Using join buffer**: 조인 버퍼를 사용함.
- **Range checked for each record**: 비효율적인 범위 체크.
- **Select tables optimized away**: 단일 로우만 필요해서 테이블 접근 생략.
- **Not Exists:**  **효율적인 안티 조인**을 수행. `LEFT JOIN`을 사용해 두 테이블을 조인. `IS NULL`을 사용하여 매칭되지 않는 데이터를 반환

`Using temporary`와 `Using filesort`는 성능 저하 가능성이 크므로 피하는 것이 좋습니다.
