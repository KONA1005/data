# SELF-CHECK 6장 문제 풀어보기

## — ERD 참고하기 —

![Untitled](/images/SELF-CHECK_6_sql/Untitled.png)

- 1번

![Untitled](/images/SELF-CHECK_6_sql/Untitled%201.png)

```sql
SELECT a.employee_id
			 , a.emp_name
			 , d.job_title
			 , b.start_date
			 , b.end_date
			 , c.department_name
FROM
		  employees a
		, job_history b
		, department c
		, jobs d
WHERE a.employee_id = b.employee_id
	AND b.department_id = c.department_id
	AND b.job_title = d.job_id
	AND a.employee_id = 101; --사번이 101 인 사원
```

1. 결과. 어떤 데이터에 해당하는지 = SELECT
2. 어디에서 = FROM ( 테이블, 뷰)
3. 어떤거? = WHERE

- 2번

![Untitled](/images/SELF-CHECK_6_sql/Untitled%202.png)

정답 )

외부 조인에서 조인조건에 데이터가 없는 테이블에만 (+) 붙인다.

and a.department_id = b.department_id(+) 로 바꿔야 한다.

- 3번

![Untitled](/images/SELF-CHECK_6_sql/Untitled%203.png)

정답)

IN은 OR로 변환할 수 있어서 외부조인 해도 값 1개인 경우 사용할 수 있다.

- 4번

![Untitled](/images/SELF-CHECK_6_sql/Untitled%204.png)

```sql
SELECT a.department_id
			, a.department_name
FROM departments a
INNER JOIN employees b
				ON ( a.department_id = b.department_id)
WHERE b.salary > 3000
ORDER BY a.department_name;

```

정답)

FROM 절에서 INNER JOIN 구문 쓴다. 조인 조건은 ON 절에 명시하고 조인 조건 외의 조건은 WHERE절에 명시한다.

- 5번

![Untitled](/images/SELF-CHECK_6_sql/Untitled%205.png)

```sql
SELECT a.department_id
			, a.department_name
FROM departments a
WHERE a.department_id In ( SELECT department_id
															FROM job_history );
```

정답)

1. 연관성 없는 쿼리 = 메인 테이블과 조인 조건이 걸리지 않는 서브쿼리
2. 연관성 있는 쿼리 → 부서 테이블의 부서 번호 = job_history 테이블의 부서번호 같은 건 조회. EXISTS 사용해 서브 쿼리 내에 조인 조건 포함
3. EXISTS = 쿼리 결과 만족하는 값이 존재하냐 ? , 참 → 메인쿼리 전체 조회결과 반환
4. 메인 테이블과 조인 조건이 걸리지 않으려면 EXISTS 조인 조건이 포함되있는 애들을 지우고 IN을사용해 일치하는게 있으면 참을 반환하는 걸로 바꿔준다.

- 6번

![Untitled](/images/SELF-CHECK_6_sql/Untitled%206.png)

```sql
SELECT 
    emp.years
    , emp.employee_id
    , emp2.emp_name
    , emp.amount_sold
FROM 
    (SELECT 
        SUBSTR(a.sales_month, 1, 4) as years
        , a.employee_id
        , SUM(a.amount_sold) AS amount_sold
     FROM 
        sales a
        , customers b 
        , countries c 
     WHERE a.cust_id = b.cust_id
        AND b.country_id = c.country_id 
        AND c.country_name = 'Italy' 
     GROUP BY SUBSTR(a.sales_month, 1, 4), a.employee_id) emp
    , (SELECT 
         years
         , MAX(amount_sold) AS max_sold
         , MIN(amount_sold) AS min_sold
         FROM (SELECT 
                 SUBSTR(a.sales_month, 1, 4) as years
                 , a.employee_id
                 , SUM(a.amount_sold) AS amount_sold
               FROM 
                sales a
                , customers b 
                , countries c 
               WHERE a.cust_id = b.cust_id
                  AND b.country_id = c.country_id 
                  AND c.country_name = 'Italy' 
               GROUP BY SUBSTR(a.sales_month, 1, 4), a.employee_id) K 
         GROUP BY years) sale
    , employees emp2
WHERE emp.years = sale.years
    AND emp.amount_sold = sale.max_sold
    AND emp.employee_id = emp2.employee_id
ORDER BY years;
```

정답)

1. 28일에 수업한 서브쿼리 내에 내용이 있었다.
- 이탈리아 고객 찾기 : customers, countries를 country_id로 조인
- 이탈리아 매출 찾기 : 위 결과와 slaes 테이블을 cust_id로 조인
- 최대매출액 구하려면 MAX함수 쓰고 연도별로 GROUP BY