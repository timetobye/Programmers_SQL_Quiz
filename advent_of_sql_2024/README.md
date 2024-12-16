Advent of SQL 2024
--------------------

# Introduction
Advent of SQL 2024에서 제공하는 SQL 문제 풀이 기록
- [Advent of SQL 2024](https://solvesql.com/collections/advent-of-sql-2024/)
- 모든 쿼리는 SQLite 기준으로 작성 되었습니다.

## 1번 - 크리스마스 게임 찾기

풀이 방법 : 패턴 매칭 이용

```sql
select game_id, name, year
from games
where 1=1
  and (name LIKE "%Christmas%" or name LIKE "%Santa%")
```

## 2번 - 펭귄 조사하기

풀이 방법 : 조건과 정렬 순서만 유의 할 것

```sql
select species, island
from penguins
group by 1, 2
order by 2, 1
```

## 3번 - 제목이 모음으로 끝나지 않는 영화

풀이 방법 : 마지막 문자열 가져오는 방법만 사용할 수 있으면 풀 수 있다.

```sql
select title
from film
where 1=1
  and rating in ("R", "NC-17")
  and substr(title, -1) not in ('A', 'E', 'I', 'O', 'U')
```

## 4번 - 지자체별 따릉이 정류소 개수 세기

풀이 방법 : 그냥 집계, group by 를 빼먹지 말자

```sql
select local, count(*) as num_stations
from station
group by 1
order by 2
```

## 5번 - 언더스코어(_)가 포함되지 않은 데이터 찾기

풀이 방법 : instr 라는 함수를 이용하면 된다.
- https://www.sqlitetutorial.net/sqlite-functions/sqlite-instr/
- instr 은 mysql 에서도 사용할 수 있다.

```sql
select distinct page_location
from ga
where 1=1
  and not instr(page_location, '_')
order by 1
```

## 6번 - 게임을 10개 이상 발매한 퍼블리셔 찾기

풀이 방법 : 집계 쿼리 응용 + 불친절한 문제 해석 하기

```sql
select name
from companies as cp
inner join 
(
  select publisher_id, count(distinct game_id) as cnt_pub_game
  from games
  group by 1
  HAVING cnt_pub_game >= 10
) as base on base.publisher_id = cp.company_id
```

## 7번 - 기증품 비율 계산하기

풀이 방법 : gift 패턴 매칭이 True 인지 여부에 대한 그룹 합계 / 전체
- instr 로 접근 하려다가 안 돼서 선회함

```sql
select round(sum(case when credit like "%gift%" then 1 else 0 end) * 100.0 / count(*), 3) as ratio
from artworks
```

## 8번 - 온라인 쇼핑몰의 월 별 매출액 집계

풀이 방법 : 패턴 매칭 + 계산
- 취소 금액은 마이너스가 적용이 되어 있어서 -1 을 안 곱해도 되더라.

```sql
select strftime('%Y-%m', order_date) as order_month
     , sum(case when order_id not like "%C%" then price * quantity else 0 end) as ordered_amount
     , sum(case when order_id like "%C%" then price * quantity else 0 end) as canceled_amount
     , sum(price * quantity) as total_amount
from orders as ord
left join (
  select order_id as o_id, price, quantity 
  from order_items
) as oi on oi.o_id = ord.order_id
group by 1
order by 1
```

## 9번 - 게임 평점 예측하기 1

풀이 방법 : 각 평점 유형 별로 나눠서 접근하면 된다.
- 테이블을 살펴볼 때 score 가 null 이면서 count 가 null 이 아닌 경우는 없어서 케이스를 줄일 수 있었다.

```sql
with omitted_base as (
      select game_id, genre_id, name, critic_score, critic_count, user_score, user_count
      from games
      where 1=1
        and (critic_score is null or user_score is null)
        and year >= 2015)
  , expert_all as (
      select genre_id
           , round(avg(critic_score), 3) as avg_cr_sc
           , ceil(avg(critic_count)) as avg_cr_cnt
      from games
      where 1=1
        and critic_score is not null
      group by 1)
  , user_all as (
      select genre_id
           , round(avg(user_score), 3) as avg_usr_sc
           , ceil(avg(user_count)) as avg_usr_cnt
      from games
      where 1=1
        and user_score is not null
      group by 1)


select game_id, name
     , IFNULL(critic_score, avg_cr_sc) as critic_score
     , IFNULL(critic_count, avg_cr_cnt) as critic_count
     , IFNULL(user_score, avg_usr_sc) as user_score
     , IFNULL(user_count, avg_usr_cnt) as user_count
from omitted_base as ob
left join expert_all as ea on ea.genre_id = ob.genre_id
left join user_all as ua on ua.genre_id = ob.genre_id
```

## 10번 - 최대값을 가진 행 찾기

풀이 방법 : 이 방법이 제일 깔끔한 방법인가? where 에 쓰는게 썩 좋아보이지는 않은데

```sql
select id
from points
where x in (select max(x) from points) or y in (select max(y) from points)
order by 1
```

## 11번 - 

풀이 방법 : 

```sql

```

## 12번 - 

풀이 방법 : 

```sql

```

## 13번 - 

풀이 방법 : 

```sql

```

## 14번 - 

풀이 방법 : 

```sql

```

## 15번 - 

풀이 방법 : 

```sql

```

## 16번 - 

풀이 방법 : 푸는 중

```sql
with origin as (
  select author, year, 1 + sum(flag) over (partition by author, cumulative_group order by year) as result
  from (
    select author, year, flag
        , sum(case when flag = 0 then 1 else 0 end) over (partition by author order by year) as cumulative_group
    from (
      select author
          , year
          , case when year = (1 + lag(year, 1) over (partition by author order by year)) then 1 else 0 end as flag
      from (select author, year from books group by 1, 2)
    ) as base
  ) as calc
) 
```

## 17번 - 

풀이 방법 : 

```sql

```

## 18번 - 

풀이 방법 : 

```sql

```

## 19번 - 

풀이 방법 : 

```sql

```

## 20번 - 

풀이 방법 : 

```sql

```

## 21번 - 

풀이 방법 : 

```sql

```

## 22번 - 

풀이 방법 : 

```sql

```

## 23번 - 

풀이 방법 : 

```sql

```

## 24번 - 

풀이 방법 : 

```sql

```
