```sql
CREATE EXTENSION IF NOT EXISTS pageinspect;

DROP TABLE IF EXISTS mvcc_erensteam;

CREATE TABLE mvcc_demo (
    id          integer PRIMARY KEY,
    name        text,
    qty         integer,
    updated_at  timestamp default now()
);

INSERT INTO mvcc_erensteam (id, name, qty)
VALUES
(1, 'Product A', 10),
(2, 'Product B', 20),
(3, 'Product C', 30);

SELECT ctid, xmin, xmax, *
FROM mvcc_erensteam
ORDER BY id;
```

<img width="742" height="211" alt="image" src="https://github.com/user-attachments/assets/47df3750-4b66-4209-9b0d-3000a8b61814" />


```sql
SELECT ctid, xmin, xmax, *
FROM mvcc_demo
WHERE id = 1;
```

<img width="717" height="105" alt="image" src="https://github.com/user-attachments/assets/4a9d6cbe-6f57-4178-88d8-9f4191c31ea9" />

xmin = satırın hangi transaction tarafından oluşturulduğu

xmax = satırın hangi transaction tarafından silindiği / eski versiyon yapıldığı

ctid = satırın fiziksel konumu


```sql
SELECT
    h.lp,
    h.t_xmin,
    h.t_xmax,
    h.t_ctid,
    h.t_infomask,
    h.t_infomask2,
    f.raw_flags,
    f.combined_flags
FROM heap_page_items(get_raw_page('mvcc_erensteam', 0)) h
CROSS JOIN LATERAL heap_tuple_infomask_flags(h.t_infomask, h.t_infomask2) f;
```

<img width="1189" height="232" alt="image" src="https://github.com/user-attachments/assets/553f2da1-c364-4c58-852a-5f3fd6e358e0" />
