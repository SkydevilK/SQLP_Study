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

## 1. Oracle

### 가. SQL 트레이스 수집

- 현재 접속해 있는 세션에만 트레이스 설정

```sql
alter sessoin set sql_trace = true;
select * from emp where DEPTNO = '10'
select * from dual;
alter session set sql_trace = true;
```

- user_dump_dest 파라미터로 지정된 서버 디렉터리 밑에 트레이스 파일(.trc) 생성

```sql
select r.value || '/' || lower(t.instance_name) || '_ora_'
from v$process p, v$session s, v$parameter r, v$instance t
where p.addr = s.paddr
and r.name = 'user_dump_dest'
and s.sid = (select sid from v$mystat where rownum = 1) ;
```

### 나. SQL 트레이스 포맷팅

> TKProf 유틸리티를 사용하면 트레이스 파일을 보기 쉽게 포맷팅해 준다.

### 다. SQL 트레이스 분석

<table>
  <thead>
    <tr>
      <th>항목</th>
      <th>설명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>call</td>
      <td>Parse: 커서를 파싱하고 실행계획을 생성하는 것의 통계<br>Execute: 커서의 실행 단계에 대한 통계<br>Fetch: 레코드를 실제로 Fetch하는 통계</td>
    </tr>
    <tr>
      <td>count</td>
      <td>Parse, Execute, Fetch 각 단계 수행 횟수</td>
    </tr>
    <tr>
      <td>cpu</td>
      <td>현재 커서가 각 단계에서 사용한 cpu time</td>
    </tr>
    <tr>
      <td>elapsed</td>
      <td>현재 커서가 각 단계를 수행하는 데 소요된 시간</td>
    </tr>
    <tr>
      <td>disk</td>
      <td>디스크로부터 읽은 블록 수</td>
    </tr>
    <tr>
      <td>query</td>
      <td>Consistent 모드에서 읽은 블록 수</td>
    </tr>
    <tr>
      <td>current</td>
      <td>Current 모드에서 읽은 블록 수</td>
    </tr>
    <tr>
      <td>rows</td>
      <td>각 단계에서 읽거나 갱신한 처리 건수</td>
    </tr>
  </tbody>
</table>

### 라. DBMS_XPLAN 패키지

<table>
  <thead>
    <tr>
      <th>DBMS_XPLAN</th>
      <th>SQL 트레이스</th>
      <th>설명</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>A-Rows</td>
      <td>rows</td>
      <td>각 단계에서 읽거나 갱신한 건수</td>
    </tr>
    <tr>
      <td>A-Time</td>
      <td>time</td>
      <td>각 단계별 소요 시간</td>
    </tr>
    <tr>
      <td>Buffers</td>
      <td>cr</td>
      <td>캐시에서 읽은 버퍼 블록 수</td>
    </tr>
    <tr>
      <td>Reads</td>
      <td>pr</td>
      <td>디스크로부터 읽은 블록 수</td>
    </tr>
  </tbody>
</table>

<table>
    <tr>
        <td align="center">query</td><td align="center">결과</td>
    </tr>
    <tr>
    <td>
        ```sql
        select *
        from table(
            DBMS_XPLAN.DISPLAY_CURSOR(null, null, 'allstats')
        );
        ```
    </td>
    <td>
        <img src="https://github.com/user-attachments/assets/82a253e8-487e-4fdd-aa6a-7de581422309"/>
    </td>
    </tr>
</table>

# 제 3절 응답 시간 분석

## 1. 대기 이벤트

> 프로세스가 일을 처리하기 위해 다른 프로세스가 마칠 때까지 기다려야 하는 상황이 발생한다.<br>
> 이때 해당 프로세스는 일을 진행할 수 있는 조건이 충족될 때까지 수면(sleep) 상태 대기한다.

### 가. 라이브러리 캐시 부하

> 라이브러리 캐시 관련 경합이 급증하면 심각한 동시성 저하 초래

- 라이브러리 캐시에서 SQL 커서를 찾고 최적화하는 과정에 경합이 발생했음을 나타내는 대기 이벤트
    - `latch: shared pool`
    - `latch: library cache`
- 수행 중인 SQL이 참조하는 오브젝트에 다른 사용자가 DDL 문장을 수행할 때 나타나는 이벤트
    - `library cache lock`
    - `library cache pin`

### 나. 데이터베이스 Call과 네트워크 부하

- 이 이벤트에 소모된 시간은 애플리케이션과 네트워크 구간에서 소모된 시간
    - `SQL*Net message from client` : 데이터베이스 경합과 상관 X, 클라이언트로부터 다음 명령이 올 때까지 Idle 상태로 기다릴 떄 발생
    - `SQL*Net message to client` : 네트워크 부하가 원인일 수 있음, 클라이언트가 바쁜 경우일 수 있음
    - `SQL*Net more data to client` : 네트워크 부하가 원인일 수 있음, 클라이언트가 바쁜 경우일 수 있음
    - `SQL*Net more data from client` : 클라이언트로부터 더 받을 데이터가 있는데 지연이 발생한 경우

### 다. 디스크 I/O 부하

- 디스크 I/O가 발생할 때마다 나타나는 대기 이벤트
    - `db file sequential read` : Single Block I/O를 수행할 때 나타나는 대기 이벤트, 인덱스 블록을 읽을 때
    - `db file scattered read` : Bultiblock I/O를 수행할 때 나타나는 대기 이벤트, Table Full Scan 또는 Index Fast Full Scan 시
    - `direct path read`
    - `direct path write`
    - `direct path write temp`
    - `direct path read temp`
    - `db file parallel read`

### 라. 버퍼 캐시 경합

> 버퍼 캐시에서 블록을 읽더라도 이들 대기 이벤트가 심하게 발생하는 순간 동시성은 현저히 저하된다.

- 버퍼 캐시에서 블록을 읽는 과정에 경합 발생 시 나타나는 대기 이벤트
    - `latch: cache buffers chains`
    - `latch: cache buffers lru chain`
    - `buffer busy waits`
    - `free buffer waits`

### 마. Lock 관련 대기 이벤트

- 'enq'로 시작되는 대기 이벤트는 Lock과 관련된 것이다.
    - `enq: TM - contention`
    - `enq: TX - row lock contention`
    - `enq: TX - index contention`
    - `enq: TX - allocate ITL entry`
    - `enq: TX - contention`
    - `latch free` : 특정 자원에 대한 래치를 여러 차례 요청했지만 해당 자원이 계속 사용 중이어서 잠시 대기 상태로 빠질 때마다 발생하는 이벤트
 
## 2. 응답 시간 분석

> Response Time = Service Time + Wait Time<br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
> = CPU Time + Queue Time

- `서비스 시간(Service Time)` : 프로세스가 정상적으로 동작하여 일을 수행한 시간(= `CPU Time`)
- `대기 시간(Wait Time)` : 프로세스가 잠시 수행을 멈추고 대기한 시간(= `Queue Time`)

## 3. AWR

> AWR(Automatic Workload Repository)은 응답 시간 분석 방법론 지원하는 Oracle의 표준 도구

```shell
@?/rdbms/adimin/awrrpt
```

### 1) 부하 프로필

<img src="https://github.com/user-attachments/assets/44d07bfa-efcb-4480-8f0f-346f2f0dbcc7"/>

- `Per Second` : 각 측정 지표 값들을 측정 시간(Snapshot Internal, 초)으로 나눔(= 초당 부하(Load) 발생량)
- `Per Transaction` : 한 트랜잭션 내에서 평균적으로 얼만큼의 부하(Load)가 발생하는지 보여줌. 트랜잭션 개수(Transactions)는 commit, rollback 수행 횟수를 더한 값

### 2) 인스턴스 효율성

<img src="https://github.com/user-attachments/assets/af143853-30ec-45c6-8d2a-2e8977d40cec"/>

- `Execute to Parse %` 항목을 제외하면 모두 100% 가까워야함
- `Parse CPU to Parse Elapsed %` 항목이 0.85%인 이유 = Active 프로세스 폭증으로 인해 과도한 Parse Call 발생(= 장애 상황)

### 3) 공유 풀 통계

<img src="https://github.com/user-attachments/assets/e65d79cc-e56e-46b4-8945-0103eec86aaa"/>

- AWR 리포트 구간 시작 시점의 공유 풀 메모리 상황과 종료 시점에서의 메모리 상황을 보여줌

### 4) 최상위 4개 대기 이벤트

<img src="https://github.com/user-attachments/assets/6074778f-c29e-4631-86e7-8b48ffdf37cf" />

- Active 프로세스 폭증으로 인해 과도한 Parse Call로 OS 레벨에서 Paging까지 심하게 발생한 장애 상황에서 측정
- CPU time은 Total Call Time에서 차지하는 비중이 가장 높아 Top 1에 위치한다면 DB의 건강상태가 양호하다는 의미
