# 제 1절 예상 실행 계획

> 실행계획이란 사용자가 요청한 SQL을 최적으로 수행하고자 DBMS 내부적으로 수립한 일련의 처리 절차이다.

## 1. Oracle

### 가. Explain Plan

> SQL을 수행하기 전에 실행 계획을 확인하고자 할 때, explain plan 명령어를 사용한다.

```sql
explain plan 
    set statement_id = 'myQuery' 
    for
        select * 
        from emp 
        where DEPTNO = '10';
```

### 나. Auto Trace

> 실행계획뿐만 아니라 여러 가지 유용한 실행 통계를 확인할 수 있다.

```sql
set autotrace on
select * from emp where DEPTNO = '10';
```

1. set autotrace on : SQL을 실제 수행하고 그 결과와 함께 실행계획 및 실행통계를 출력한다.
2. set autotrace on explain : SQL을 실제 수행하고 그 결과와 함께 실행계획을 출력한다.
3. set autotrace on statistics : SQL을 실제 수행하고 그 결과와 함께 실행통계를 출력한다.
4. set autotrace traceonly : SQL을 실제 수행하지만 그 결과는 출력하지 않고 실행계획과 통계만을 출력한다.
5. set autotrace traceonly explain : SQL을 실제 수행하지 않고 실행계획만을 출력한다.
6. set autotrace traceonly statistics : SQL을 실제 수행하지만 그 결과는 출력하지 않고 실행통계만을 출력한다.

### 다. DBMS_XPLAN 패키지

> plan_table에 저장된 실행계획을 좀 더 쉽게 출력해 볼 수 있게 한다.

## 2. SQL Server

```sql
set shoplan_text on
select * from dbo.emp where DEPTNO = '10'
```

- 쿼리는 실제 실행하지 않는다.

# 제 2절 SQL 트레이스

