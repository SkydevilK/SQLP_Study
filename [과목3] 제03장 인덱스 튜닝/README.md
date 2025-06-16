# 제 1절 인덱스 기본 원리

## 1. 인덱스 구조

### 가. 인덱스 기본 구조

<img src="https://github.com/user-attachments/assets/ce0a904f-c0f3-484f-a680-95414b8de2c8" />

- `index height` : 루트에서 리프 블록까지의 거리
- 루트와 브랜치 블록은 각 하위 노드들의 데이터 값 범위를 나타내는 키 값과 그 키 값에 해당하는 블록을 찾는데 필요한 주소 정보를 가진다.
- 리프 블록은 인덱스 키 값과 그 키 값에 해당하는 테이블 레코드를 찾아가는 데 필요한 주소 정보(ROWID)를 가진다.
  - 키 값이 같을 떄는 ROWID 순으로 정렬된다.
- 리프 블록은 항상 인덱스 키(Key) 값으로 정렬돼 있기 때문에 '범위 스캔(Range Scan, 검색 조건에 해당하는 범위만 읽다가 멈추는 것)'이 가능하고, 정방향(Ascending)과 역방향(Descending) 스캔이 둘 다 가능하도록 양방향 연결 리스트(Double Linked list) 구조로 연결돼 있다.
- Oracle에선 인덱스 구성 컬럼이 모두 Null인 레코드는 인덱스에 저장하지 않는다.
- SQL Server에선 인덱스 구성 컬럼이 모두 null인 레코드도 인덱스 저장한다.
- null 값을 Oracle은 맨 뒤에 저장하고 SQL Server는 맨 앞에 저장한다.

### 나. 인덱스 탐색

> 인덱스 탐색과정은 수직적 탐색과 수평적 탐색으로 나눌 수 있다.

- `수직적 탐색` : 수평적 탐ㅁ색을 위한 시작 지점을 찾는 과정, 루트에서 리프 블록까지 아래쪽으로 진행된다.
- `수평적 탐색` : 인덱스 리프 블록에 저장된 레코드끼리 연결된 순서에 따라 좌에서 우 또는 우에서 좌로 스캔한다.

## 2. 다양한 인덱스 스캔 방식

### 가. Index Range Scan

> 인덱스 루트 블록에서 리프 블록까지 수직적으로 탐색한 후에 리프 블록을 필요한 범위(Range)만 스캔하는 방식
<img src="https://github.com/user-attachments/assets/7e889ce7-fddf-4188-acf9-08fef7b9f6d2" width='500'/>
- B*Tree 인덱스의 가장 일반적이고 정상적인 형태의 엑세스 방식

### 나. Index Full Scan

> 수직적 탐색없이 인덱스 리프 블록을 처음부터 끝까지 수평적으로 탐색하는 방식<br>대개는 데이터 검색을 위한 최적의 인덱스가 없을 때 차선으로 선택
<img src="https://github.com/user-attachments/assets/c133c601-20f6-4517-a0e8-86d468a9c2dc" width='500'/>

#### Index Full Scan의 효용성

- 인덱스 선두 컬럼(ename)이 조건절에 없으면 옵티마이저는 우선적으로 Table Full Scan을 고려한다.
- 데이터 저장 공간 = 컬럼 길이 * 레코드수 &rarr; 인덱스가 차지하는 면적은 테이블보다 훨씩 적다.
- index full scan 효율 사례 = 연봉이 5000 초과 하는 사원이 극히 일부인 경우
- index full scan 비효율 사례 = 연봉이 1000 초과 하는 사원(대부분의 사원 연봉이 1000 초과) &rarr; 거의 모든 record table access 발생(table full scan, index range scan 보다 비효율)
  - order by, min/max 목적이라면 index full scan이 더 효율
- index full scan 통한 쿼리는 index column 순으로 정렬 &rarr; order by, min/max 생략 가능

### 다. Index Unique Scan

> 수직적 탐색만으로 데이터를 찾는 스캔 방식<br>Unique 인덱스를 '=' 조건으로 탐색하는 경우에 작동한다.
<img src="https://github.com/user-attachments/assets/ee55a0eb-12fd-4df6-8350-28b32ab637bd" width='500'/>

### 라. Index Skip Scan

> Oracle에서 인덱스 선두 컬럼이 조건절에 빠졌어도 인덱스를 활용하는 새로운 스캔방식
<img src="https://github.com/user-attachments/assets/aca1fbf4-6b21-4de9-b36a-771f8b175927" width='500'/>
- 인덱스 선두 칼럼이 조건절로 사용되지 않으면 옵티마이저는 기본적으로 Table Full Scan을 선택한다.
- Table Full Scan보다 I/O를 줄일 수 있거나 정렬된 결과를 쉽게 얻을 수 있다면 Index Full Scan 방식을 사용한다.
- 수행 원리
  - 루트 또는 브랜치 블록에서 읽은 칼럼 값 정보를 이용해 조건에 부합하는 레코드를 포함할 '가능성이 있는' 하위 블록(브랜치 또는 리프 블록)만 골라서 액세스하는 방식
- 조건절에 빠진 인덱스 선두 칼럼의 Distinct Value 개수가 적고 후행 칼럼의 Distinct Value 개수가 많을 때 유용하다.

### 마. Index Fast Full Scan

> Index Fast Full Scan은 Index Full Scan보다 빠르다.<br>
> 인덱스 트리 구조를 무시하고 인덱스 세그먼트 전체를 Multiblock Read 방식으로 스캔하기 때문이다.
<img src="https://github.com/user-attachments/assets/1bd7101b-8401-4403-98f7-b5e4ad51b1c1" />

- `Singleblock Read` : 한번의 I/O Call에 하나의 데이터 블록만 읽어 메모리에 적재 &rarr; db file sequential read 
- `Multiblock Read` : I/O Call 시점에 인접한 블록들을 같이 읽어 메모리에 적재 &rarr; db file scattered read
  - `인접한 블록` : 한 extent 내에 속한 블록

### 바. Index Range Scan Descending

> Index Range Scan과 기본적으로 동일한 스캔 방식<br> 인덱스를 뒤에서 앞쪽으로 스캔하기 때문에 내림차순으로 정렬된 결과를 얻는다.
<img src="https://github.com/user-attachments/assets/0630fc75-e3a2-4e6d-8a54-1e29302c2c2f" width='500'/>

## 3. 인덱스 종류

### 가. B*Tree 인덱스

> 모든 DBMS가 B*Tree 인덱스를 기본적으로 제공한다.

#### 1) Unblanced Index

> 다른 leaf node에 비해 root block 거리가 더 멀거나 가까운 leaf node 발생 가능
<img src="https://github.com/user-attachments/assets/c8f3372e-6bca-4131-b19a-33356f4d8a44"/>

- delete 동작으로 발생 가능
- B*Tree 구조에선 절대 발생하지 않는다.
  - 'B'는 'Balanced'의 약자로서, 인덱스 루트에서 리프 블록까지 어떤 값으로 탐색하더라도 읽는 블록 수가 같다.
  - 루트로부터 모든 리프 블록까지의 높이(Height)가 동일하다.

#### 2) Index Skew

> 인덱스 엔트리가 왼쪽 또는 오른쪽에 치우치는 현상

<img src="https://github.com/user-attachments/assets/63fbc547-aac4-43b6-802e-c39c846410f9"/>

- 텅 빈 index block은 commit 하는 순간 freelist로 반환되지만 index 구조는 그대로 존재
- 상위 block에서 leaf block을 가리키는 entry가 그대로 남아있어 재사용 가능
- 새로운 값이 입력되기 전 다른 node에 index 분할이 발생되면 재사용 가능 &rarr; 상위 block에서 leaf block을 가리키는 entry가 제거돼 다른 branch 자식 node로 이동하고 freelist에서 제거<br>
&rArr; record가 모두 삭제된 block은 재사용 가능하지만, 다시 채워질때까지 index scan 효율 &darr; &darr;

#### 3) Index Sparse

> 인덱스 블록 전반에 걸쳐 밀도(density)가 떨어지는 현상

<img src="https://github.com/user-attachments/assets/507094a8-0f3d-4f16-b734-c18501e2644c"/>

- delete 이후 index block density 50% ex) 100만건 중 50만건 지우더라도 index block 수 = 2001개
- index scan 효율 ↓
- 지워진 자리에 새로운 값이 입력되지 않으면 영영 재사용 안될 수 있음 → leaf block 가리키는 entry 삭제 불가능 ⇒ record 수가 일정한데도 index 공간 사용량이 계속 커질 수 있음


#### 4) 인덱스 재생성

> Fragmentation(파편화) 때문에 인덱스 크기가 계속 증가하고 스캔 효율이 나빠지면 인덱스를 재생성하거나 DBMS가 제공하는 명령어를 이용해 빈 공간 제거를 한다.

- 인덱스 분할에 의한 경합이 현저히 높을 때
- 자주 사용되는 인덱스 스캔 효율을 높이고자 할 때. 특히 NL 조인에서 반복 액세스 되는 인덱스 높이(height)가 증가했을 때
- 대량의 delete 작업을 수행한 이후 다시 레코드가 입력되기까지 오랜 기간이 소요될 때
- 총 레코드 수가 일정한데도 인덱스가 계속 커질 때

### 나. 비트맵 인덱스

<img src="https://github.com/user-attachments/assets/0a4f05c1-f324-42c6-8998-fa1115a80261" />

- distinct value 개수가 적을 때 효율 ↓ → 다양한 조건절이 사용되는 쿼리에 유리
- B*Tree index보다 훨씬 적은 용량 차지 → index가 여러개 필요한 대용량 테이블에 유리
- distinct value 개수가 많으면 B*Tree index 보다 공간 많이 차지
- Lock에 의한 DML 부하 심함 → record 하나만 변경되더라도 비트맵 범위에 속한 모든 레코드 lock
- 읽기 위주의 대용량 데이터 웨어하우스(OLAP) 환경에 적합

### 다. 함수기반 인덱스

> 데이터 입력, 수정 시 함수를 적용해야 하므로 다소 부하가 발생할 수 있다.<br>
> 사용된 함수가 사용자 정의 함수일 때는 부하가 더 심하다.

### 라. 리버스 키 인덱스

> index 값을 거꾸로 변환하여 저장하는 인덱스
<img src="https://github.com/user-attachments/assets/6625633e-4546-4245-8de0-93904241d95b" />

```sql
create index 주문_idx01 on 주문( reverse(주문일시) )
```

- 일련번호나 주문일시 같은 컬럼에 인덱스를 만들면, 입력되는 값이 순차적으로 증가하여 가장 오른쪽 리프 블록에만 데이터가 쌓인다. = right growing(= right hand) index
- 동시 insert가 심할 때 index race가 일어나 tps &darr;
- reverse key index를 통해 데이터 고르게 분포
- index range scan 불가능 ∵ 데이터를 거꾸로 입력하여 = 조건으로만 검색 가능

### 마. 클러스터 인덱스

> 클러스터 키 값이 같은 레코드가 한 블록에 모이도록 저장

<img src="https://github.com/user-attachments/assets/2304c789-4b22-491d-a1c6-b816ee03edba" />

- index leaf block = data record
- 정렬 상태를 유지하며 저장
