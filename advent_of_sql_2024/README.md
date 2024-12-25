Advent of SQL 2024
--------------------

# Introduction
Advent of SQL 2024에서 제공하는 SQL 문제 풀이 기록
- [Advent of SQL 2024](https://solvesql.com/collections/advent-of-sql-2024/)
- 모든 쿼리는 SQLite 기준으로 작성 되었습니다.

[alt text](advent_of_sql_Image.png)

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

## 11번 - 서울숲 요일별 대기오염도 계산하기

풀이 방법 : 함수하고 조건에 맞추면 크게 문제 없는데 정렬 순서는 어거지로 한 것 같다.

```sql
select 
     case when strftime('%w', measured_at) = '1' then '월요일'
          when strftime('%w', measured_at) = '2' then '화요일'
          when strftime('%w', measured_at) = '3' then '수요일'
          when strftime('%w', measured_at) = '4' then '목요일'
          when strftime('%w', measured_at) = '5' then '금요일'
          when strftime('%w', measured_at) = '6' then '토요일'
          when strftime('%w', measured_at) = '0' then '일요일'
      end as weekday
     , round(avg(no2), 4) as no2
     , round(avg(o3), 4) as o3
     , round(avg(co), 4) as co
     , round(avg(so2), 4) as so2
     , round(avg(pm10), 4) as pm10
     , round(avg(pm2_5), 4) as pm2_5
from measurements
group by 1
order by case when strftime('%w', measured_at) = '1' then 1
          when strftime('%w', measured_at) = '2' then 2
          when strftime('%w', measured_at) = '3' then 3
          when strftime('%w', measured_at) = '4' then 4
          when strftime('%w', measured_at) = '5' then 5
          when strftime('%w', measured_at) = '6' then 6
          when strftime('%w', measured_at) = '0' then 7
      end
```

## 12번 - 3년간 들어온 소장품 집계하기

풀이 방법 : year 추출하는 방법만 알면 해결 가능

```sql
select classification
    , sum(case when strftime('%Y', acquisition_date) = '2014' then 1 else 0 end) as '2014'
    , sum(case when strftime('%Y', acquisition_date) = '2015' then 1 else 0 end) as '2015'
    , sum(case when strftime('%Y', acquisition_date) = '2016' then 1 else 0 end) as '2016'
from artworks
where 1=1
group by 1
order by 1
```

## 13번 - 게임 개발사의 주력 플랫폼 찾기

풀이 방법 : 조인 잘 하고 dense_rank 이용해서 처리 하면 됨

```sql
select cp_name as developer, pl_name as platform, sum_sales_total as sales
from (
  select cp_name, pl_name, sum_sales_total, dense_rank() over(partition by cp_name order by sum_sales_total desc) as rank
  from (
    select cp_name, pl_name, sum(sales_total) as sum_sales_total
    from (
      select developer_id as dev_id, cp.name as cp_name
           , pl.name as pl_name, gm.platform_id as gm_pl_id
           , (sales_eu + sales_jp + sales_na + sales_other) as sales_total
      from games as gm
      left join companies as cp on cp.company_id = gm.developer_id
      left join platforms as pl on pl.platform_id = gm.platform_id
      where 1=1
        and developer_id is not NULL
    ) as base
    group by 1, 2
  ) as calc
) as res
where 1=1
  and rank = 1
```

## 14번 - 전력 소비량 이동 평균 구하기

풀이 방법 : 이동 평균 구하는 것은 어렵지 않은데 오래간만에 rows between ~ and ~ 구문을 사용해서 헷갈렸다.
- 좀 더 깔끔하게 풀 수 있을 것 같은데 그냥 넘어감

```sql
select *
from (
  select measured_at as end_at
      , round(avg(zone_quads) over (rows BETWEEN 6 PRECEDING and 1 PRECEDING), 2) as zone_quads
      , round(avg(zone_smir) over (rows BETWEEN 6 PRECEDING and 1 PRECEDING), 2) as zone_smir
      , round(avg(zone_boussafou) over (rows BETWEEN 6 PRECEDING and 1 PRECEDING), 2) as zone_boussafou
  from power_consumptions
  where 1=1
    and measured_at BETWEEN '2017-01-01' and '2017-02-01T00:10:00'
) as base
where zone_quads is not null
```

## 15번 - 폐쇄할 따릉이 정류소 찾기 2

풀이 방법 : 그냥 어거지로 푼 듯. 왜 맞았는지는 좀 애매함

```sql
with rental_history_bs as (
      select * 
      from rental_history
      where 1=1
        and (
          (rent_at >= '2018-10-01' and rent_at < '2018-11-01') or 
          (rent_at >= '2019-10-01' and rent_at < '2019-11-01') or
          (return_at >= '2018-10-01' and return_at < '2018-11-01') or 
          (return_at >= '2019-10-01' and return_at < '2019-11-01') 
          )
    ),
    calc_rh as (
      select st_id, dt, r_type, sum(cnt) as total_sum
      from (
          select rent_station_id as st_id
              , strftime('%Y-%m', rent_at) dt
              , 'rent' as r_type
              , count(*) as cnt
          from rental_history_bs
          group by 1, 2, 3

          union

          select return_station_id as st_id
              , strftime('%Y-%m', return_at) as dt
              , 'return' as r_type
              , count(*) as cnt
          from rental_history_bs
          group by 1, 2, 3
      ) as base
      group by 1, 2, 3
    )


select st_id as station_id, name, local, usage_pct
from (
  select st_id
      , round(100.0 * sum(case when dt='2019-10' then total_sum else 0 end) / sum(case when dt='2018-10' then total_sum else 0 end), 2) as usage_pct
      , name, local
  from calc_rh
  left join station as st on st.station_id = calc_rh.st_id
  where 1=1
    and st_id in (select st_id from calc_rh group by 1 having count(*) >= 2)
  group by 1
) res
where usage_pct <= 50 and usage_pct > 0
```

## 16번 - 스테디셀러 작가 찾기

풀이 방법
- 소설 작가
- encounter 방식으로 그룹을 만들어서 처리함

```sql
with origin as (
  select author, year, flag, cumulative_group, 1 + sum(flag) over (partition by author, cumulative_group order by year) as result
  from (
    select author, year, flag
        , sum(case when flag = 0 then 1 else 0 end) over (partition by author order by year) as cumulative_group
    from (
      select author, year
           , case when year = (1 + lag(year, 1) over (partition by author order by year)) then 1 else 0 end as flag
      from (select author, year from books where genre='Fiction' group by 1, 2)
    ) as base
  ) as calc
) 

select author, max(year) as year, max(result) as depth
from (
  select * 
  from origin 
  where result >= 5
) 
group by 1
order by 1
```

## 17번 - 멀티 플랫폼 게임 찾기

풀이 방법 : 조건에 맞게 그룹화 하여 붙이면 된다.
- 문제는 문제 설명과 테이블의 데이터가 어긋나 있는 부분이 있는데 기준을 어디에 두느냐에 따라 완전히 다른 결과를 얻게 될 수도 있다.
- 다소 아쉬운 문제

```sql
select name
from (
  select name, count(distinct mj_pl_name) as cnt_mj_pl_name
  from games as gm
  inner join (
    select platform_id as pl_id, name as pl_name
         , case when name in ('PS3', 'PS4', 'PSP', 'PSV') then 'Sony'
                when name in ('Wii', 'WiiU', 'DS', '3DS') then 'Nintendo'
                else 'Microsoft'
            end as mj_pl_name
    from platforms
    where 1=1
      and name in ('PS3', 'PS4', 'PSP', 'PSV', 'Wii', 'WiiU', 'DS', '3DS', 'X360', 'XONE')
  ) as pl on pl.pl_id = gm.platform_id
  where 1=1
    and year >= 2012
  group by 1
  HAVING count(distinct mj_pl_name) >= 2
) as base
```

## 18번 - 펭귄 날개와 몸무게의 상관 계수

풀이 방법 : 그냥 상관 계수를 쿼리로 구현하면 된다.

```sql
with calc as (
      select species
          , x_avg_x * y_avg_y as value_1
          , x_avg_x * x_avg_x as value_2
          , y_avg_y * y_avg_y as value_3
      from (
        select species
            , flipper_length_mm
            , avg(flipper_length_mm) over (partition by species) as avg_flnm
            , flipper_length_mm - avg(flipper_length_mm) over (partition by species) as x_avg_x
            , body_mass_g
            , avg(body_mass_g) over (partition by species) as avg_bmg
            , body_mass_g - avg(body_mass_g) over (partition by species) as y_avg_y
        from penguins
        where 1=1
          and flipper_length_mm is not null or body_mass_g is not null
      ) as base
)
select species
     , round(sum(value_1) / sqrt(sum(value_2)* sum(value_3)), 3) as corr
from calc
group by 1
```

## 19번 - 전국 카페 주소 데이터 정제하기

풀이 방법 : sqlite 니까 이렇게 푼 것 같다. 다른 서비스였으면 그냥 split(' ')으로 해결 했을듯

```sql
select
  SUBSTR(address, 1, INSTR(address, ' ') - 1) as sido,
  SUBSTR(address, INSTR(address, ' ') + 1, INSTR(SUBSTR(address, INSTR(address, ' ') + 1), ' ') - 1) as sigungu,
  count(*) as cnt
from cafes
group by 1, 2
order by 3 desc
```

## 20번 - 미세먼지 수치의 계절간 차이

풀이 방법 : case when 으로 처리 가능

```sql
select
  case when measured_at >= '2022-03-01' and measured_at < '2022-06-01' then 'spring'
       when measured_at >= '2022-06-01' and measured_at < '2022-09-01' then 'summer'
       when measured_at >= '2022-09-01' and measured_at < '2022-12-01' then 'autumn'
       else 'winter'
   end as 'season',
   median(pm10) as pm10_median,
   round(avg(pm10), 2) as pm10_average
from measurements
group by 1
```

## 21번 - 세션 유지 시간을 10분으로 재정의하기

풀이 방법 : lag 써서 풀이 하면 된다. 시간 제약이 10분 이니까 10분 미만으로 처리

```sql
select user_pseudo_id, event_timestamp_kst, event_name, ga_session_id
     , sum(flag) over(order by event_timestamp_kst) as new_session_id
from (
  select user_pseudo_id
      , event_timestamp_kst, lag_et_kst, event_name, ga_session_id
      , strftime('%s', event_timestamp_kst) - strftime('%s', lag_et_kst) as second_interval
      , case when strftime('%s', event_timestamp_kst) - strftime('%s', lag_et_kst) < 600 then 0 else 1 end as flag
  from (
    select user_pseudo_id
        , event_timestamp_kst, event_name, ga_session_id
        , IFNULL(lag(event_timestamp_kst) over (order by event_timestamp_kst asc), 0) as lag_et_kst
    from ga
    where 1=1
      and user_pseudo_id = 'a8Xu9GO6TB'
  ) as base
) as calc
```

## 22번 - 친구 수 집계하기

풀이 방법 : 테이블을 보니까 단방향이어서 기준을 하나로 놓고 집계 할 수 있도록 테이블을 합쳤다.

```sql
select user_id, ifnull(num_friends, 0) as num_friends
from users as usr
left join (
  select user_a_id as u_id, count(*) as num_friends
  from (
    select user_a_id, user_b_id
    from edges

    union

    select user_b_id, user_a_id
    from edges
  ) as base
  group by 1
) as calc on calc.u_id = usr.user_id
order by 2 desc
```

## 23번 - 유량(Flow)와 저량(Stock)

풀이 방법 : window 함수 잘 쓰기

```sql
select ac_year as 'Acquisition year', cnt as 'New acquisitions this year (Flow)'
     , sum(cnt) over(order by ac_year asc) as 'Total collection size (Stock)'
from (
  select strftime('%Y', acquisition_date) as ac_year
      , count(*) as cnt
  from artworks
  where 1=1
    and acquisition_date is not null
  group by 1
) as base
order by 1
```

## 24번 - 세 명이 서로 친구인 관계 찾기

풀이 방법 : 맞기는 했는데 좀 어거지로 푼 느낌이 있다. 최적화 방법이 있을듯

```sql
select 
user_a_id, usr_a_id as user_b_id, usr_b_id as user_c_id
from (
  select user_a_id, usr_a_id, usr_b_id
  from edges as ed_a
  left join (
    select user_a_id as usr_a_id
        , user_b_id as usr_b_id
    from edges
  ) as ed_b on ed_b.usr_a_id = ed_a.user_b_id
) as base
left join (
  select user_a_id as us_a_id
      , user_b_id as us_c_id
  from edges
) as net on net.us_c_id = base.usr_b_id
-- where user_a_id = 3820
where 1=1
  and user_a_id < usr_a_id and usr_a_id < usr_b_id and user_a_id = us_a_id
  and (user_a_id = 3820 or usr_a_id = 3820 or usr_b_id = 3820)
limit 1000
```

## 25번 - 메리 크리스마스 2024

풀이 방법 : 
- 다 풀고 나니 크리스마스에 이걸 푼 내가 레전드

```sql
select 'Merry Christmas!'
```
