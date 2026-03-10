Bu, mağazada pahalı ürünleri / pahalı oyunları filtreleme senaryosu. (Это фильтр дорогих товаров в магазине.)
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM products
WHERE price > 60;
```
-без индексов
<img width="809" height="239" alt="1 1" src="https://github.com/user-attachments/assets/77ecf50e-aa95-4e34-85b9-936353bc3146" />


-B-tree
<img width="916" height="377" alt="2 1" src="https://github.com/user-attachments/assets/edff58cd-507e-4982-a911-dd123700b845" />


Bu da mağazada ucuz ürünleri / indirimli ucuz oyunları arama senaryosu.(Это фильтр дешёвых товаров.)
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM products
WHERE price < 10;
```
без индексов
<img width="814" height="190" alt="1 2" src="https://github.com/user-attachments/assets/e36b328d-9aa8-453a-93d2-ead7ba9d8a3d" />


B-tree
<img width="913" height="335" alt="2 2" src="https://github.com/user-attachments/assets/ed1ae773-df76-4b03-b6f0-c5f1f55f34b9" />


Bu, tek bir ödeme işlemini bulma senaryosu.(Это поиск одной конкретной транзакции по её идентификатору.)
transaction ID
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM payments
WHERE provider_txn_id = 'TXN-150000';
```
без индексов
<img width="815" height="401" alt="1 3" src="https://github.com/user-attachments/assets/b2391a6a-5d13-455b-aff0-944db18f91b8" />


B-tree
<img width="918" height="218" alt="2 3" src="https://github.com/user-attachments/assets/7f98b2a4-1ed9-4776-9f15-7bec341f0876" />


Hash
<img width="1003" height="307" alt="image" src="https://github.com/user-attachments/assets/dce58443-25ad-4550-9b7d-0821e5f352aa" />

Bu, mağazada ürün adıyla prefix arama senaryosu.(Это поиск по началу названия продукта.)
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM products
WHERE name LIKE 'Product 12%';
```
без индексов
<img width="810" height="356" alt="1 4" src="https://github.com/user-attachments/assets/20077bb3-1c40-4144-9ec5-30ffbbc430ca" />


B-tree
<img width="914" height="303" alt="2 4" src="https://github.com/user-attachments/assets/b4c22be0-f235-4ff4-afc8-17954a62c0d9" />

Bu, mağazada birkaç yayıncıya göre filtreleme senaryosu.(Это фильтрация товаров по нескольким издателям.)
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM products
WHERE publisher_name IN
('Top Publisher 1', 'Top Publisher 2', 'Top Publisher 3', 'Top Publisher 4', 'Top Publisher 5');
```
без индексов
<img width="915" height="251" alt="1 5" src="https://github.com/user-attachments/assets/a9cc9705-5869-42bf-87d9-e1fe82415295" />


B-tree
<img width="919" height="267" alt="2 5" src="https://github.com/user-attachments/assets/7594d144-94e5-4527-9866-a17460d70f2b" />


Hash
<img width="917" height="323" alt="image" src="https://github.com/user-attachments/assets/874a4ba3-219e-4236-8c37-8a8d8c33cdef" />


Bu, mağazada açıklama içinde anahtar kelime arama senaryosu.(Это поиск слова внутри текста описания.)
```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM products
WHERE description LIKE '%story%';
```
без индексов
<img width="915" height="351" alt="1 6" src="https://github.com/user-attachments/assets/2ece9d39-2881-4829-92ec-29dc7387e2db" />


B-tree
<img width="923" height="287" alt="2 6" src="https://github.com/user-attachments/assets/48458252-5a24-4a59-b7c9-c02e8830b94d" />
