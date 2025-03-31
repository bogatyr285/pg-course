 ### Заливаем тайские перевозки
Объем порядка 6 млн.строк (600МБ):
wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql -U postgres < thai.sql


### Считаем количество поездок
```
thai=# select count(*) from book.ride;
 count
--------
 144000
(1 row)
```