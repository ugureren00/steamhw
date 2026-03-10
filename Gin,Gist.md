Gin

GIN-1 — array contains
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT product_id, name, tags
FROM products
WHERE tags @> ARRAY['action','singleplayer'];
```
<img width="947" height="293" alt="image" src="https://github.com/user-attachments/assets/7db371a0-77fd-4e4c-bfe4-337ddc021c12" />

GIN-2 — jsonb contains
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT product_id, name, metadata_json
FROM products
WHERE metadata_json @> '{"platform":"windows","cloud_save":true}'::jsonb;
```
<img width="946" height="293" alt="image" src="https://github.com/user-attachments/assets/245e723e-9234-458e-ad22-0a56c1d6baa0" />

GIN-3 — jsonb contains on users
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT user_id, username, preferences_json
FROM users
WHERE preferences_json @> '{"ui_language":"tr","newsletter":true}'::jsonb;
```
<img width="943" height="316" alt="image" src="https://github.com/user-attachments/assets/18f83398-cb8f-4c0a-bdf6-47de3f5a8c8b" />

GIN-4 — full text search
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT product_id, name
FROM products
WHERE to_tsvector('english', coalesce(description, ''))
      @@ plainto_tsquery('english', 'gameplay technical');
```
<img width="946" height="336" alt="image" src="https://github.com/user-attachments/assets/5c6f914d-8d1f-46dc-a4ca-1ad6843f10d6" />

GIN-5 — trigram / substring search
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT product_id, name
FROM products
WHERE description ILIKE '%gameplay%';
```
<img width="946" height="240" alt="image" src="https://github.com/user-attachments/assets/4d78447a-c002-4e2f-b119-ecd015e5abb1" />

Gist

GiST-1 — point containment
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT product_id, name, sale_period
FROM products
WHERE sale_period @> TIMESTAMP '2026-03-15 12:00:00';
```
<img width="947" height="242" alt="image" src="https://github.com/user-attachments/assets/3cd5a486-8379-4481-8e50-236da143e146" />


GiST-2 — overlap
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT product_id, name, sale_period
FROM products
WHERE sale_period && tsrange(
    TIMESTAMP '2026-03-10 00:00:00',
    TIMESTAMP '2026-03-20 00:00:00',
    '[]'
);
```
<img width="947" height="246" alt="image" src="https://github.com/user-attachments/assets/d7e84dc9-5768-4bff-8cbb-148c60a9a240" />


GiST-3 — refund window contains point
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT order_id, user_id, refund_period
FROM orders
WHERE refund_period @> TIMESTAMP '2026-03-18 10:00:00';
```
<img width="948" height="319" alt="image" src="https://github.com/user-attachments/assets/2f7c45c7-1da8-4376-8b29-0e7cccb97afd" />


GiST-4 — refund window overlap
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT order_id, user_id, refund_period
FROM orders
WHERE refund_period @> TIMESTAMP '2026-03-18 10:00:00';
```
<img width="954" height="289" alt="image" src="https://github.com/user-attachments/assets/94ff734c-9820-4343-ae1e-fc6351ba7775" />

GiST-5 — games active period overlap
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT game_id, title, active_period
FROM games
WHERE active_period && daterange(
    DATE '2025-01-01',
    DATE '2026-12-31',
    '[]'
);
```
<img width="957" height="351" alt="image" src="https://github.com/user-attachments/assets/ffd1cca3-b013-4d28-ac58-2e1911cf0695" />



JOIN-1 — users + orders

1.<img width="988" height="317" alt="image" src="https://github.com/user-attachments/assets/4c1a0c01-aee7-4729-bf54-0313834d2fdb" />

```sql
"Limit  (cost=20943.27..20943.29 rows=7 width=53) (actual time=163.317..163.322 rows=11 loops=1)"
"  Buffers: shared hit=1187"
"  ->  Sort  (cost=20943.27..20943.29 rows=7 width=53) (actual time=163.316..163.319 rows=11 loops=1)"
"        Sort Key: o.order_date DESC"
"        Sort Method: quicksort  Memory: 26kB"
"        Buffers: shared hit=1187"
"        ->  Nested Loop  (cost=20934.80..20943.17 rows=7 width=53) (actual time=163.285..163.304 rows=11 loops=1)"
"              Buffers: shared hit=1187"
"              ->  Nested Loop  (cost=20934.38..20942.41 rows=1 width=26) (actual time=163.275..163.277 rows=1 loops=1)"
"                    Buffers: shared hit=1173"
"                    ->  Limit  (cost=20933.96..20933.96 rows=1 width=12) (actual time=163.207..163.209 rows=1 loops=1)"
"                          Buffers: shared hit=1169"
"                          ->  Sort  (cost=20933.96..21433.96 rows=200000 width=12) (actual time=163.206..163.207 rows=1 loops=1)"
"                                Sort Key: (count(*)) DESC"
"                                Sort Method: top-N heapsort  Memory: 25kB"
"                                Buffers: shared hit=1169"
"                                ->  GroupAggregate  (cost=3.14..19933.96 rows=200000 width=12) (actual time=0.033..147.968 rows=110000 loops=1)"
"                                      Group Key: u_1.user_id"
"                                      Buffers: shared hit=1169"
"                                      ->  Merge Join  (cost=3.14..16433.96 rows=300000 width=4) (actual time=0.024..104.247 rows=300000 loops=1)"
"                                            Merge Cond: (u_1.user_id = o_1.user_id)"
"                                            Buffers: shared hit=1169"
"                                            ->  Index Only Scan using users_pkey on users u_1  (cost=0.42..5204.42 rows=200000 width=4) (actual time=0.014..23.179 rows=200000 loops=1)"
"                                                  Heap Fetches: 0"
"                                                  Buffers: shared hit=550"
"                                            ->  Index Only Scan using idx_orders_user on orders o_1  (cost=0.42..6980.42 rows=300000 width=4) (actual time=0.007..28.435 rows=300000 loops=1)"
"                                                  Heap Fetches: 0"
"                                                  Buffers: shared hit=619"
"                    ->  Index Scan using users_pkey on users u  (cost=0.42..8.44 rows=1 width=22) (actual time=0.062..0.062 rows=1 loops=1)"
"                          Index Cond: (user_id = u_1.user_id)"
"                          Buffers: shared hit=4"
"              ->  Index Scan using idx_orders_user on orders o  (cost=0.42..0.69 rows=7 width=35) (actual time=0.008..0.020 rows=11 loops=1)"
"                    Index Cond: (user_id = u.user_id)"
"                    Buffers: shared hit=14"
"Planning:"
"  Buffers: shared hit=32"
"Planning Time: 0.505 ms"
"Execution Time: 163.384 ms"
```

JOIN-2 — orders + payments
2.<img width="1338" height="623" alt="image" src="https://github.com/user-attachments/assets/5272a034-9b77-44be-a75d-84ae15ef9603" />

```sql
"Limit  (cost=20110.74..20113.07 rows=20 width=62) (actual time=154.933..160.501 rows=20 loops=1)"
"  Buffers: shared hit=3522 read=8380"
"  ->  Gather Merge  (cost=20110.74..45373.63 rows=216524 width=62) (actual time=154.931..160.494 rows=20 loops=1)"
"        Workers Planned: 2"
"        Workers Launched: 2"
"        Buffers: shared hit=3522 read=8380"
"        ->  Sort  (cost=19110.71..19381.37 rows=108262 width=62) (actual time=150.415..150.419 rows=20 loops=3)"
"              Sort Key: o.order_date DESC"
"              Sort Method: top-N heapsort  Memory: 27kB"
"              Buffers: shared hit=3522 read=8380"
"              Worker 0:  Sort Method: top-N heapsort  Memory: 27kB"
"              Worker 1:  Sort Method: top-N heapsort  Memory: 27kB"
"              ->  Parallel Hash Join  (cost=9733.77..16229.90 rows=108262 width=62) (actual time=67.461..127.049 rows=86111 loops=3)"
"                    Hash Cond: (p.order_id = o.order_id)"
"                    Buffers: shared hit=3450 read=8380"
"                    ->  Parallel Seq Scan on payments p  (cost=0.00..6168.00 rows=125000 width=35) (actual time=0.053..15.248 rows=100000 loops=3)"
"                          Buffers: shared hit=96 read=4822"
"                    ->  Parallel Hash  (cost=8380.50..8380.50 rows=108262 width=31) (actual time=66.506..66.506 rows=86111 loops=3)"
"                          Buckets: 262144  Batches: 1  Memory Usage: 20160kB"
"                          Buffers: shared hit=3260 read=3558"
"                          ->  Parallel Seq Scan on orders o  (cost=0.00..8380.50 rows=108262 width=31) (actual time=0.022..26.383 rows=86111 loops=3)"
"                                Filter: (status = ANY ('{completed,refunded}'::text[]))"
"                                Rows Removed by Filter: 13889"
"                                Buffers: shared hit=3260 read=3558"
"Planning:"
"  Buffers: shared hit=8"
"Planning Time: 0.257 ms"
"Execution Time: 160.541 ms"
```


JOIN-3 — games + products
3. <img width="1358" height="655" alt="image" src="https://github.com/user-attachments/assets/40662770-6c1b-4e90-8638-768e80e95359" />

```sql
"Limit  (cost=7.05..155.94 rows=25 width=89) (actual time=0.199..0.204 rows=25 loops=1)"
"  Buffers: shared hit=54"
"  ->  Incremental Sort  (cost=7.05..50540.46 rows=8485 width=89) (actual time=0.198..0.200 rows=25 loops=1)"
"        Sort Key: g.game_id, p.product_id"
"        Presorted Key: g.game_id"
"        Full-sort Groups: 1  Sort Method: quicksort  Average Memory: 28kB  Peak Memory: 28kB"
"        Buffers: shared hit=54"
"        ->  Nested Loop  (cost=1.13..50158.64 rows=8485 width=89) (actual time=0.048..0.160 rows=30 loops=1)"
"              Buffers: shared hit=54"
"              ->  Nested Loop  (cost=0.71..12922.45 rows=4073 width=35) (actual time=0.039..0.066 rows=3 loops=1)"
"                    Buffers: shared hit=15"
"                    ->  Index Scan using games_pkey on games g  (cost=0.42..11672.96 rows=4073 width=20) (actual time=0.010..0.027 rows=3 loops=1)"
"                          Index Cond: ((game_id >= 1000) AND (game_id <= 5000))"
"                          Buffers: shared hit=6"
"                    ->  Memoize  (cost=0.29..0.43 rows=1 width=19) (actual time=0.011..0.011 rows=1 loops=3)"
"                          Cache Key: g.developer_id"
"                          Cache Mode: logical"
"                          Hits: 0  Misses: 3  Evictions: 0  Overflows: 0  Memory Usage: 1kB"
"                          Buffers: shared hit=9"
"                          ->  Index Scan using developers_pkey on developers d  (cost=0.28..0.42 rows=1 width=19) (actual time=0.009..0.009 rows=1 loops=3)"
"                                Index Cond: (developer_id = g.developer_id)"
"                                Buffers: shared hit=9"
"              ->  Index Scan using idx_products_game_id on products p  (cost=0.42..9.06 rows=8 width=58) (actual time=0.004..0.027 rows=10 loops=3)"
"                    Index Cond: (game_id = g.game_id)"
"                    Buffers: shared hit=39"
"Planning:"
"  Buffers: shared hit=26"
"Planning Time: 0.547 ms"
"Execution Time: 0.249 ms"
```


JOIN-4 — users + libraries + products

4. <img width="1313" height="587" alt="image" src="https://github.com/user-attachments/assets/4016d746-8797-45c7-910a-7606ff246aa8" />

```sql
"Limit  (cost=5246.25..5264.46 rows=20 width=62) (actual time=24.403..29.733 rows=20 loops=1)"
"  Buffers: shared hit=2844 read=1"
"  ->  Nested Loop  (cost=5246.25..7775.48 rows=2779 width=62) (actual time=24.402..29.728 rows=20 loops=1)"
"        Buffers: shared hit=2844 read=1"
"        ->  Gather Merge  (cost=5245.83..5562.56 rows=2779 width=42) (actual time=24.381..27.292 rows=20 loops=1)"
"              Workers Planned: 1"
"              Workers Launched: 1"
"              Buffers: shared hit=2765"
"              ->  Sort  (cost=4245.82..4249.91 rows=1635 width=42) (actual time=21.506..21.565 rows=454 loops=2)"
"                    Sort Key: l.playtime_minutes DESC"
"                    Sort Method: quicksort  Memory: 1067kB"
"                    Buffers: shared hit=2765"
"                    Worker 0:  Sort Method: quicksort  Memory: 602kB"
"                    ->  Hash Join  (cost=356.00..4158.56 rows=1635 width=42) (actual time=1.694..18.656 rows=7600 loops=2)"
"                          Hash Cond: (l.user_id = u.user_id)"
"                          Buffers: shared hit=2758"
"                          ->  Parallel Seq Scan on libraries l  (cost=0.00..3595.29 rows=78956 width=31) (actual time=0.010..9.392 rows=67190 loops=2)"
"                                Filter: (playtime_minutes > 1000)"
"                                Rows Removed by Filter: 16810"
"                                Buffers: shared hit=2360"
"                          ->  Hash  (cost=304.24..304.24 rows=4141 width=15) (actual time=1.604..1.605 rows=4001 loops=2)"
"                                Buckets: 8192  Batches: 1  Memory Usage: 244kB"
"                                Buffers: shared hit=351"
"                                ->  Index Scan using users_pkey on users u  (cost=0.42..304.24 rows=4141 width=15) (actual time=0.027..0.952 rows=4001 loops=2)"
"                                      Index Cond: ((user_id >= 1000) AND (user_id <= 5000))"
"                                      Buffers: shared hit=351"
"        ->  Index Scan using products_pkey on products p  (cost=0.42..0.80 rows=1 width=24) (actual time=0.121..0.121 rows=1 loops=20)"
"              Index Cond: (product_id = l.product_id)"
"              Buffers: shared hit=79 read=1"
"Planning:"
"  Buffers: shared hit=29"
"Planning Time: 0.474 ms"
"Execution Time: 29.776 ms"
```


JOIN-5 — products + reviews + users

5. <img width="1179" height="625" alt="image" src="https://github.com/user-attachments/assets/0752a309-b357-4cb3-9911-e5043f5ef746" />

```sql
"Limit  (cost=64347.01..64347.06 rows=20 width=75) (actual time=400.180..403.305 rows=20 loops=1)"
"  Buffers: shared hit=597 read=27242, temp read=911 written=914"
"  ->  Sort  (cost=64347.01..64734.58 rows=155027 width=75) (actual time=400.178..403.301 rows=20 loops=1)"
"        Sort Key: (count(*)) DESC, (round(avg(r.rating), 2)) DESC"
"        Sort Method: top-N heapsort  Memory: 27kB"
"        Buffers: shared hit=597 read=27242, temp read=911 written=914"
"        ->  Finalize GroupAggregate  (cost=39754.91..60221.79 rows=155027 width=75) (actual time=277.997..383.748 rows=73502 loops=1)"
"              Group Key: p.product_id"
"              Buffers: shared hit=597 read=27242, temp read=911 written=914"
"              ->  Gather Merge  (cost=39754.91..56281.51 rows=129190 width=75) (actual time=277.984..322.778 rows=73502 loops=1)"
"                    Workers Planned: 2"
"                    Workers Launched: 2"
"                    Buffers: shared hit=597 read=27242, temp read=911 written=914"
"                    ->  Partial GroupAggregate  (cost=38754.88..40369.76 rows=64595 width=75) (actual time=273.126..296.499 rows=24501 loops=3)"
"                          Group Key: p.product_id"
"                          Buffers: shared hit=597 read=27242, temp read=911 written=914"
"                          ->  Sort  (cost=38754.88..38916.37 rows=64595 width=31) (actual time=273.106..279.220 rows=58889 loops=3)"
"                                Sort Key: p.product_id"
"                                Sort Method: external merge  Disk: 2552kB"
"                                Buffers: shared hit=597 read=27242, temp read=911 written=914"
"                                Worker 0:  Sort Method: external merge  Disk: 2368kB"
"                                Worker 1:  Sort Method: external merge  Disk: 2368kB"
"                                ->  Parallel Hash Join  (cost=20539.58..33594.02 rows=64595 width=31) (actual time=149.652..252.602 rows=58889 loops=3)"
"                                      Hash Cond: (p.product_id = r.product_id)"
"                                      Buffers: shared hit=525 read=27242"
"                                      ->  Parallel Seq Scan on products p  (cost=0.00..12394.67 rows=104167 width=19) (actual time=0.050..58.735 rows=83334 loops=3)"
"                                            Buffers: shared hit=148 read=11205"
"                                      ->  Parallel Hash  (cost=19732.14..19732.14 rows=64595 width=16) (actual time=148.678..148.895 rows=58889 loops=3)"
"                                            Buckets: 262144  Batches: 1  Memory Usage: 11808kB"
"                                            Buffers: shared hit=355 read=16037"
"                                            ->  Parallel Hash Join  (cost=9973.39..19732.14 rows=64595 width=16) (actual time=35.405..111.040 rows=58889 loops=3)"
"                                                  Hash Cond: (r.user_id = u.user_id)"
"                                                  Buffers: shared hit=355 read=16037"
"                                                  ->  Parallel Seq Scan on reviews r  (cost=0.00..9528.08 rows=87872 width=20) (actual time=0.037..43.782 rows=70000 loops=3)"
"                                                        Filter: (visibility_status = 'public'::text)"
"                                                        Rows Removed by Filter: 13333"
"                                                        Buffers: shared hit=96 read=8130"
"                                                  ->  Parallel Hash  (cost=9207.67..9207.67 rows=61258 width=4) (actual time=34.629..34.630 rows=48889 loops=3)"
"                                                        Buckets: 262144  Batches: 1  Memory Usage: 7840kB"
"                                                        Buffers: shared hit=259 read=7907"
"                                                        ->  Parallel Seq Scan on users u  (cost=0.00..9207.67 rows=61258 width=4) (actual time=0.019..19.979 rows=48889 loops=3)"
"                                                              Filter: (account_status = 'active'::text)"
"                                                              Rows Removed by Filter: 17778"
"                                                              Buffers: shared hit=259 read=7907"
"Planning:"
"  Buffers: shared hit=35"
"Planning Time: 0.723 ms"
"Execution Time: 403.895 ms"
```


Select <img width="1835" height="564" alt="image" src="https://github.com/user-attachments/assets/5d4ee6ed-6178-43e9-94ed-74d26d5fcc12" />

Insert <img width="1830" height="580" alt="image" src="https://github.com/user-attachments/assets/324174f2-86b8-4de2-82e7-27ab4332c24d" />

Delete <img width="1838" height="592" alt="image" src="https://github.com/user-attachments/assets/b17b0483-8972-4f5a-94cc-dc039f378c8e" />

Cpu <img width="1814" height="542" alt="image" src="https://github.com/user-attachments/assets/619febdc-91de-4dc6-8cf9-ec1ae016ee62" />




