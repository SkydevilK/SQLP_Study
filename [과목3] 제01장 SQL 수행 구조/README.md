# 제 1절 데이터베이스 아키텍처

## 1. 데이터 베이스 구조

### 가. Oracle 구조

> Oracle에서는 디스크에 저장된 데이터 집합(Datafile, Redo Log File, Control File 등)을 데이터베이스(Database)라고 부른다.<br>
> SGA 공유 메모리 영역과 이를 액세스하는 프로세스 집합을 합쳐서 인스턴스 (Instance)라고 부른다

<table>
<tr>
<td>
<div align="center">
    <img src="https://github.com/user-attachments/assets/d98bef63-2bb8-4179-b5c8-3cc9abd3af31" width="400"/>
</div>
<td>
<div align="center">
  <img src="https://github.com/user-attachments/assets/1a5c138e-2863-426b-b812-012e1ac39812" width="400"/>
</div>
</td>
</tr>
</table>

- 기본적으로 하나의 인스턴스가 하나의 데이터베이스만 액세스 하는 것이 원칙
- RAC(Real Applicatoin Cluster) 환경에서는 여러 인스턴스가 하나의 데이터베이스를 액세스할 수 있다.
- **하나의 인스턴스가 여러 데이터베이스를 액세스 할 수는 없다.**

### 나. SQL Server 구조

> SQL Server는 하나의 인스턴스당 최고 32,767개의 데이터베이스를 정의해 사용할 수 있다.<br>
> master, model, msdb, tempdb 등의 시스템 데이터베이스가 만들어지며, 사용자 데이터베이스를 추가로 생성한다.

- 데이터베이스 하나를 만들 때마다 주(Primary 또는 Main) 데이터 파일과 트랜잭션 로그 파일이 하나씩 생긴다. 전자는 확장자가 mdf이고 후자는 ldf이다.
- 저장할 데이터가 많으면 보조(Non-Primary) 데이터 파일을 추가할 수 있으며 확장자는 ndf이다.

## 2. 프로세스

### 가. 서버 프로세스

> 사용자 프로세스와 통신하면서 사용자의 각종 명령을 처리하며, SQL Server에선 Worker 쓰레드가 같은 역할을 담당한다.

- SQL 파싱 및 최적화 수행
  - 블록을 읽어 데이터를 정렬해 결과 집합을 만들고 네트워크를 통해 전송하는 작업
  - 스스로 처리되지 않은 기능(데이터 파일로부터 DB 버퍼 캐시로 블록을 적재하거나 Dirty 블록을 캐시에서 밀어냄으로써 Free 블록을 확보하는 일, Redo 로그 버퍼를 비우는 일 등은 OS와 I/O 서브시스템, 백그라운드 프로세스가 대신 처리하도록 시스템 Call을 통해 요청한다.
 
#### 전용 서버 방식

<div align="center">
<img src="https://github.com/user-attachments/assets/c85c4710-2e32-4f29-aead-31f5f77f7100"/>
</div>

- SQL을 수행할 때마다 연결 요청을 반복하면 서버 프로세스의 생성과 해제도 반복하게 되므로 DBMS에 부하가 생김
- 전용 서버 방식을 사용하는 OLTP성 애플리케이션에선 Connection Pooling 기법을 필수적으로 사용해야 함

#### 공유 서버 방식

<div align="center">
<img src="https://github.com/user-attachments/assets/85a9ab7b-864f-485e-8adf-e35433d818b2"/>
</div>

> 하나의 서버 프로세스를 여러 사용자 세션이 공유하는 방식

### 나. 백그라운드 프로세스

<table>
<tr>
<td align="center" colspan="3">백그라운드 프로세스</td>
</tr>
<tr>
<td align="center">Oracle</td><td align="center">SQL Server</td><td align="center">설명</td>
</tr>
<tr>
<td align="center">System Monitor<br>(SMON)</td>
<td align="center">Database cleanup/<br>shrinking thread</td>
<td>
장애가 발생한 시스템을 재기동할 때 인스턴스 복구를 수행하고,<br>임시 세그먼트와 익스텐트 모니터링
</td>
</tr>
<tr>
<td align="center">Process Monitor<br>(PMON)</td>
<td align="center">Open Data Services<br>(OPS)</td>
<td>
이상이 생긴 프로세스가 사용하던 리소스 복구
</td>
</tr>
<tr>
<td align="center">Database Writers<br>(DBWn)</td>
<td align="center">Lazywriter thread</td>
<td>
버퍼 캐시에 있는 Dirty 버퍼를 데이터 파일에 기록
</td>
</tr>
<tr>
<td align="center">Log Writer<br>(LGWR)</td>
<td align="center">Log writer thread</td>
<td>
로그 버퍼 엔트리를 Redo 로그 파일에 기록
</td>
</tr>
<tr>
<td align="center">Archiver<br>(ARCn)</td>
<td align="center">N/A</td>
<td>
꽉 찬 Redo 로그가 덮어 쓰여지기 전에 Archive 로그 디렉토리로 백업
</td>
</tr>
<tr>
<td align="center">Checkpoint<br>(CKPT)</td>
<td align="center">Database Checkpoint thread</td>
<td>
이전에 Checkpoint가 일어났던 마지막 시점 이후의 DB 변경 사항을 기록하도록 트리거링하고,<br> 기록이 완료되면 현재 어디까지 기록했는지를 컨트롤 파일과 데이터 파일 헤더에 저장
</td>
</tr>
<tr>
<td align="center">Recoverer<br>(RECO)</td>
<td align="center">Distributed Transaction Coordinator(DTC)</td>
<td>
분산 트랜잭션 과정에 발생한 문제 해결
</td>
</tr>
</table>

## 3. 데이터 저장 구조

### 가. 데이터 파일

<div align="center">
<img src="https://github.com/user-attachments/assets/1a2de830-7829-4bfa-a88c-328f2ef3479a"/>
</div>

> Oracle과 SQL Server 모두 물리적으로는 데이터 파일에 데이터를 저장하고 관리한다.<br>
> 공간을 할당하고 관리하기 위한 논리적인 구조도 크게 다르지 않지만 약간의 차이가 있다.

#### 1) 블록(=페이지)

- 대부분 DBMS에서 I/O는 블록 단위로 이뤄진다. 데이터를 읽고 쓸 때의 논리적인 단위가 블록이다.
  - Oracle은 '블록(Blcok)'이라고 하고, SQL Server는 '페이지(Page)'라고 한다.
  - Oracle은 2KB, 4KB, 8KB, 16KB, 32KB의 다양한 블록 크기를 사용할 수 있다.
  - SQL Server는 단일 크기인 8KB를 사용한다.
- SQL의 성능을 좌우하는 가장 중요한 성능지표는 액세스하는 블록 개수이며, 옵티마이저의 판단에 가장 큰 영향을 미치는 것도 액세스해야 할 블록 개수이다.
  - ex) 인덱스를 이용해 테이블을 액세스할지 Full Table Scan 할 지 결정하는 데 있어 가장 큰 기준은 읽어야 할 레코드 수가 아닌 **읽어야 할 블록 개수**이다.

#### 2) 익스텐트

- 테이블스페이스로부터 공간을 할당하는 단위는 익스텐트(Extend)이다.
- 익스텐트 내 블록은 논리적으로 인접하지만, 익스텐트끼리 서로 인접하지 않는다.

#### 3) 세그먼트

- SQL Server에서는 세그먼트라는 용어를 사용하지 않지만, 힙 구조 또는 인덱스 구조의 오브젝트가 여기에 속한다.
- 세그먼트는 테이블, 인덱스, Undo처럼 저장공간을 필요로 하는 데이터베이스 오브젝트이다.
  - 한 개 이상의 익스텐트를 사용한다.
  - 테이블을 생성할 때, 내부적으로 테이블 세그먼트가 생성된다.
  - 인덱스를 생성할 때, 내부적으로 인덱스 세그먼트가 생성된다.
- 다른 오브젝트는 세그먼트와 1:1 대응 관계를 갖지만 파티션은 1:M 관계를 갖는다.
- 한 세그먼트는 자신이 속한 테이블스페이스 내 여러 데이터 파일에 걸쳐 저장될 수 있다.

#### 4) 테이블스페이스

- 세그먼트를 담는 컨테이너로서, 여러 데이터 파일로 구성된다.
  - SQL Server의 파일 그룹이 Oracle 테이블스페이스에 해당한다.
- 사용자는 세그먼트를 위한 테이블스페이스를 지정한다.
- DBMS는 실제 값을 저장할 데이터 파일을 선택하고 익스텐트를 할당한다.
- 각 세그먼트는 정확히 한 테이블스페이스에만 속한다. 한 테이블스페이스에는 여러 세그먼트가 존재할 수 있다.

<img src="https://github.com/user-attachments/assets/50621013-b83e-4283-b312-6e984ca95917" />
<img src="https://github.com/user-attachments/assets/9ccc179d-2840-4398-a7da-f49e0ac7c820" />

### 나. 임시 데이터 파일

- 대량의 정렬이나 해시 작업을 수행하다가 메모리 공간이 부족해지면 중간 결과 집합을 저장하는 용도
- Redo 정보를 생성하지 않기 때문에 복구 불가 및 백업할 필요 없음
- Oracle에선 임시 테이블스페이스를 여러 개 생성해 두고, 사용자마다 별도의 임시 테이블스페이스를 지정할 수 있다.
- SQL Server는 단 하나의 tempdb 데이터베이스를 사용한다.
  - 전역 리소스로서 시스템에 연결된 모든 사용자의 임시 데이터를 저장한다.

### 다. 로그 파일

- Redo 로그(= SQL Server 트랜잭션 로그)는 DB 버퍼 캐시에 가해지는 모든 변경 사항을 기록하는 파일이다.
- Fast Commit = 사용자 변경 내용이 메모리상의 버퍼 블록에만 기록되고 아직 디스크엔 기록되지 않았더라도 Redo 로그를 믿고 빠르게 커밋
- 변경된 메모리 버퍼 블록을 디스크 상의 데이터 블록에 기록하는 작업 = Random I/O 방식 &rarr; 속도 &darr;
- log 기록 = Append 방식<br>
&rArr; 로그 파일에 Append 방식으로 빠르게 기록하고 버퍼 블록과 데이터 파일 간 동기화(DBWR, Checkpoint)는 이후 배치(Batch) 방식으로 일괄 처리

#### Online Redo 로그(Oracle)

- 마지막 체크포인트 이후부터 사고 발생 직전까지 수행됐던 트랜잭션들을 Redo 로그를 이용해 재현 &rarr; '캐시 복구'
- 최소 두 개 이상의 파일로 구성, 현재 사용중인 파일이 꽉 차면 다음 파일로 로그 스위칭(log switching)이 발생, 계속 로그를 써 나가다가 모든 파일이 꽉 차면 다시 첫 번째 파일부터 사용하는 라운드 로빈(round-robin) 방식 사용

#### 트랜잭션 로그(SQL Server)

- 데이터베이스마다 트랜잭션 로그 파일이 하나씩 생기며 확장자는 ldf
- 내부적으로 '가상 로그 파일' 세그먼트로 나뉘며 개수에 대한 옵션 지정 필요

#### Archived(=Offline) Redo 로그(Oracle)

- Online Redo 로그가 재사용되기 전에 다른 위치로 백업해 둔 파일
- 디스크가 깨지는 등 물리적인 저장 매체에 문제가 생겼을 때 데이터베이스 복구를 위해 사용

## 4. 메모리 구조

> 메모리 구조는 시스템 공유 메모리 영역과 프로세스 메모리 영역으로 구분된다.

### 시스템 공유 메모리 영역

> 여러 프로세스(또는 쓰레드)가 동시에 액세스할 수 있는 메모리 영역<br>
> Oracle에선 System Global Area(SGA), SQL Server에선 Memory Pool
- DB 버퍼 캐시, 공유 풀, 로그 버퍼
- 시스템 구조와 제어 구조를 캐싱하는 영역

### 프로세스 전용 메모리 영역

> 데이터를 정렬하고 세션과 커서 상태 정보 저장 용도<br>
> Process Global Area(PGA)

### 가. DB 버퍼 캐시

> 데이터 파일로부터 읽어 들인 데이터 블록을 담는 캐시 영역<br>
> 인스턴스에 접속한 모든 사용자 프로세스는 서버 프로세스를 통해 DB 버퍼 캐시의 버퍼 블록을 동시에(내부적으로는 버퍼 Lock을 통해 직렬화) 액세스할 수 있다.

- 대부분(Direct Path Read 매커니즘 제외) 읽고자 하는 블록을 먼저 버퍼 캐시에서 찾아보고 없을 때 디스크에서 읽는다.
- 디스크 조회 시에도 버퍼 캐시에 적재한 후 읽는다.
- 디스크 I/O는 물리적으로 액세스 암(Arm)이 움직이면서 헤드를 통해 동작 &rarr; 속도 &darr;
- 메모리 I/O는 전기적 신호 &rarr; 속도 &uarr;

#### 1) 버퍼 블록의 상태

- `free buffer` : 아직 데이터가 읽히지 않아 비어있는 상태(Clean 버퍼)이거나, 동기화돼 있는 상태여서 데이터가 언제든지 덮어써도 무방한 버퍼 블록
- `dirty buffer` : 배퍼에 캐시된 이후 변경이 발생했지만 아직 디스크에 기록되지 않아 데이터 파일 블록과 동기화가 필요한 버퍼 블록
- `pinned buffer` : 읽기 또는 쓰기 작업이 현재 진행 중인 버퍼 블록

#### 2) LRU 알고리즘

- 사용빈도가 높은 데이터 블록 위주로 버퍼 캐시가 구성되도록 LRU(least recently used) 알고리즘을 사용
- 모든 버퍼 블록 헤더를 LRU 체인에 연결해 사용빈도 순으로 위치를 옮겨가다가, Free 버퍼가 필요해질 때 액세스 빈도가 낮은 쪽(LRU end) 데이터 블록부터 밀어내는 방식

### 나. 공유 풀

> 공유 풀(Shared Pool)은 딕셔너리 캐시와 라이브러리 캐시로 구성된다.<br>
> LRU 알고리즘을 사용하며 SQL Server에서는 프로시저 캐시(Procedure Cache)라고 부른다.

- `데이터베이스 딕셔너리(Dictionary)`는 테이블, 인덱스 같은 오브젝트는 물론 테이블스페이스, 데이터 파일, 세그먼트, 익스텐트, 사용자, 제약에 관한 메타 정보를 저장하는 곳
- `라이브러리 캐시`는 사용자가 수행한 SQL 문과 실행계획, 저장 프로시저를 저장해 두는 캐시영역이다

### 다. 로그 버퍼

> 로그 엔트리를 파일에 곧바로 기록하는 것이 아닌 로그 버퍼(Log Buffer)에 기록한다.

#### 라. PGA

> 각 Oracle 서버 프로세스는 자신만의 PGA(Process, Program, Private Global Area) 메모리 영역을 할당받고, 이를 프로세스에 종속적인 고유 데이터를 저장하는 용도로 사용한다.<br>
> PGA는 다른 프로세스와 공유되지 않는 독립적인 메모리 공간으로서, 래치 매커니즘이 필요 없어 SGA 버퍼 캐시에서 읽는 것보다 훨씬 빠르다.

- `User Global Area(UGA)` : 하나의 프로세스가 여러 개 세션을 위해 일하기 위해 각 세션을 위한 독립적인 메모리 공간
- `Call Global Area(CGA)` : 하나의 데이터베이스 Call을 넘어서 다음 Call까지 계속 참조돼야 하는 정보는 UGA, Call이 진행되는 동안에만 필요한 데이터는 CGA에 담는다.
- `Sort Area` : 데이터 정렬을 위해 사용되는 Sort Area는 소트 오퍼레이션이 진행되는 동안 공간이 부족해질 때마다 청크(Chunk) 단위로 조금씩 할당된다.

# 제 2절 SQL 처리 과정

## 1. 구조적, 집합적, 선언적 질의 언어

> SQL의 정의
> 원하는 결과 집합을 구조적, 집합적으로 선언하지만, 그 결과 집합을 만드는 과정은 절차적일 수밖에 없다.<br>
> 프로시저가 필요한데, 그런 프로시저를 만들어 내는 DBMS 내부 엔진이 바로 SQL 옵티마이저다.

## 2. SQL 처리 과정

<div align="center">
    <img src="https://github.com/user-attachments/assets/711eca69-bc94-460f-9ff8-a910655301db"/>
</div>
   
<table>
    <tr>
        <td align="center" colspan="2">엔진</td><td align="center">역할</td>
    </tr>
    <tr>
        <td align="center" colspan="2">Parser</td>
        <td>SQL 문장의 개별 구성요소를 분석하고 파싱해서 파싱 트리(내부적인 구조체)를 생성<br>SQL의 문법적 오류가 없는지(Syntax 체크), 의미상 오류가 없는지(Semantic 체크) 확인</td>
    </tr>
    <tr>
        <td align="center" rowspan="3">Optimizer</td>
        <td align="center">Query Transformer</td>
        <td>파싱된 SQL을 좀 더 일반적이고 표준적인 형태로 반환</td>
    </tr>
    <tr>
        <td align="center">Estimator</td>
        <td>쿼리 수행의 각 단계의 선택도, 카디널리티, 비용을 계산 &rarr; 실행계획 전체에 대한 총 비용 계산</td>
    </tr>
    <tr>
        <td align="center">Plan Generator</td>
        <td>쿼리 수행 시 후보군이 될만한 실행계획들을 생성</td>
    </tr>
    <tr>
        <td colspan="2" align="center">Row-Source Generator</td>
        <td>옵티마이저가 생성한 실행계획을 SQL 엔진이 실행할 수 있는 코드(또는 프로시저) 형태로 변환</td>
    </tr>
    <tr>
        <td colspan="2" align="center">SQL Engine</td>
        <td>SQL 실행</td>
    </tr>
</table>

## 3. SQL 옵티마이저

> 사용자가 원하는 작업을 가장 효율적으로 수행할 수 있는 최적의 데이터 액세스 경로를 선택해 주는 DBMS의 핵심 엔진

1. 사용자로부터 전달받은 쿼리를 수행하는 데 후보군이 될만한 실행계획들을 찾아낸다.
2. 데이터 딕셔너리(Data Dictionary)에 미리 수집해 둔 오브젝트 통계 및 시스템 통계정보를 이용해 각 실행계획의 예상비용을 산정한다.
3. 최저 비용을 나타내는 실행계획을 선택한다.

## 4. 실행계획과 비용

> SQL 옵티마이저가 생성한 처리절차를 사용자가 확인할 수 있도록 다음과 같이 트리 구조로 표현한 것이 실행계획이다.

## 5. 옵티마이저 힌트

> 개발자가 직접 인덱스를 지정하거나 조인 방식을 변경함으로써 더 좋은 실행계획으로 유도하는 매커니즘

### 힌트가 무시되는 경우

- 문법적 오류
- 의미적으로 안 맞게 힌트를 기술
    - ex) 서브쿼리에 unnest와 push_subq를 같이 기술한 경우 (unnest되지 않은 서브쿼리만이 push_subq 힌트의 적용 대상)
- 잘못된 참조 사용
    - ex) 없는 테이블이나 별칭(Alias)을 사용하거나 없는 인덱스명을 지정한 경우 등
- 논리적으로 불가능한 액세스 경로
    - ex) 조인절에 등치(=) 조건이 하나도 없는데 해시 조인으로 유도, null 허용칼럼에 대한 인덱스를 이용해 전체 건수를 세려고 시도하는 등

### Oracle 힌트 종류
<table>
    <tr>
        <th>분류</th>
        <th>힌트</th>
    </tr>
    <tr>
        <td>최적화 목표</td>
        <td>all_rows<br>first_rows(n)</td>
    </tr>
    <tr>
    <td>액세스 경로</td>
    <td>full<br>cluster<br>hash<br>index, no_index<br>index_asc, index_desc<br>index_combine<br>index_join<br>index_ffs, no_index_ffs<br>index_ss, no_index_ss<br>index_ss_asc, index_ss_desc</td>
    </tr>
    <tr>
        <td>쿼리 변환</td>
        <td>no_query_transformation<br>use_concat<br>no_expand<br>rewrite, no_rewirte<br>merge, no_merge<br>star_transformation, no_star_transformation<br>fact, no_fact<br>unnest, no_unnest</td>
    </tr>
    <tr>
        <td>조인 순서</td>
        <td>ordered<br>leading</td>
    </tr>
    <tr>
        <td>조인 방식</td>
        <td>use_nl, no_use_nl<br>use_nl_with_index<br>use_merge, no_use_merge<br>use_hash, no_use_hash</td>
    </tr>
    <tr>
        <td>병렬 처리</td>
        <td>parallel, no_parallel<br>pq_distribute<br>parallel_index, no_parallel_index</td>
    </tr>
    <tr>
        <td>기타</td>
        <td>append, noappend<br>cache, nocache<br>push_pred, no_push_pred<br>push_subq,no_push_subq<br>qb_name<br>cursor_sharing_exact<br>driving_site<br>dynamic_sampling<br>model_min_analysis</td>
    </tr>
</table>

# 제 3절 데이터베이스 I/O 매커니즘

## 1. 블록 단위 I/O

> SQL 성능을 좌우하는 가장 중요한 성능지표는 액세스하는 블록 개수이다.<br>
> 옵티마이저의 판단에 가장 큰 영향을 미치는 것도 액세스해야 할 블록 개수이다.

- 데이터 파일에서 DB 버퍼 캐시로 블록을 적재할 때
- 데이터 파일에서 블록을 직접 읽고 쓸 때
- 버퍼 캐시에서 블록을 읽고 쓸 때
- 버퍼 캐시에서 변경된 블록을 다시 데이터 파일에 쓸 때

## 2. 메모리 I/O vs 디스크 I/O

### 가. I/O 효율화 튜닝의 필요성

> 디스크를 경유한 데이터 입출력은 디스크의 액세스 암(Arm)이 움직이면서 헤드를 통해 데이터를 읽고 쓰기 때문에 느린 반면,<br>
> 메모리를 통한 입출력은 전기적 신호에 불과하기 때문에 디스크를 통한 I/O에 비해 매우 빠르다.

- 메모리는 물리적으로 한정된 자원이므로, 디스크 I/O를 최소화하고 버퍼 캐시 효율을 높이는 것이 데이터베이스 I/O 튜닝의 목표

### 나. 버퍼 캐시 히트율(BCHR, Buffer Cache Hit Ratio)

> 버퍼 캐시 효율을 측정하는 지표<br>
> 전체 읽은 불록 중에서 메모리 버퍼 캐시에서 찾은 비율<br>
> BCHR = (버퍼 캐시에서 곧바로 찾은 블록 수 / 총 읽은 블록 수) X 100

### 다. 네트워크, 파일시스템 캐시가 I/O 효율에 미치는 영향

- 네트워크 전송량에 따른 성능 저하
- 논리적인 블록 요청 횟수를 최소화 하도록 튜닝

## 3. Sequential I/O vs Random I/O

<div align="center">
    <img src="https://github.com/user-attachments/assets/e7a8aef4-5b2b-448c-b1b1-355c79cc392b"/>
</div>

- `시퀀셜 액세스` : 차례대로 읽어 나가는 방식(1 ~ 5)
- `랜덤 액세스` : 한 건을 읽기 위해 한 블록씩 접근(4, 6)

## 4. Single Block I/O vs MultiBlock I/O

- `Single Block I/O` : 한 번의 I/O Call에 하나의 데이터 블록만 읽어 메모리에 적재하는 방식
    - 인덱스를 통해 테이블을 액세스할 때 인덱스와 테이블 블록 모두 이 방식을 사용
- `MultiBlock I/O` : I/O Call이 필요한 시점에 인접한 블록들을 같이 읽어 메모리에 적재하는 방식
    - '인접한 블록' : 한 익스텐트(Extent)내에 속한 블록
 
## 5. I/O 효율화 원리

> 논리적인 I/O 요청 횟수를 최소화하는 것이 I/O 효율화 튜닝의 핵심 원리

- 필요한 최소 블록만 읽도록 SQL 작성
- 최적의 옵티마이징 팩터 제공
- 옵티마이저 힌트를 사용해 최적의 액세스 경로로 유도
