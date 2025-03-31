## query:
```
explain ANALYZE
WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
order_place AS (
    SELECT count(t.id) as order_place, t.fkride
    FROM book.tickets t
    group by t.fkride
)
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,
      t.order_place, st.all_place
FROM book.ride r
JOIN book.schedule as s
      on r.fkschedule = s.id
JOIN book.busroute br
      on s.fkroute = br.id
JOIN book.busstation bs
      on br.fkbusstationfrom = bs.id
JOIN order_place t
      on t.fkride = r.id
JOIN all_place st
      on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;
```

## no indexes:
```
   Functions: 80
   Options: Inlining false, Optimization false, Expressions true, Deforming true
   Timing: Generation 8.396 ms, Inlining 0.000 ms, Optimization 2.253 ms, Emission 58.475 ms, Total 69.124 ms
 Execution Time: 1885.586 ms
 ```


## index book.busstation(city):
```` 
  Planning Time: 0.562 ms
 JIT:
   Functions: 57
   Options: Inlining true, Optimization true, Expressions true, Deforming true
   Timing: Generation 1.891 ms, Inlining 10.675 ms, Optimization 170.637 ms, Emission 116.180 ms, Total 299.383 ms
 Execution Time: 3530.264 ms
 ````