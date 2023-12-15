# Задача №1
В условии нам дано следующее:

**По названию поставщика выдать названия всех товаров, 
поставленные (R) данным поставщиком, с указанием для каждого 
товара даты последней поставки. Использовать явный курсор.**

Будем писать процедуру по структуре, которая уже описана в документе ["Как писать процедуры?"](https://github.com/NikitaBogoslovskiy/DatabaseCourse/tree/main/lab09/how-to-create-procedures.md). 

Навигация по решению:

1. [Настройка процедуры](#настройка_процедуры)
2. [Написание логики](#написание_логики)
    1. [Проверка входных параметров](#проверка_входных_параметров)
    2. [Решение основной задачи](#решение_основной_задачи)
    3. [Вывод результирующей таблицы](#вывод_результирующей_таблицы)
3. [Итоговый код процедуры](#итоговый_код_процедуры)

<a name="настройка_процедуры"></a> 
## Настройка процедуры
1. Дадим процедуре имя `get_goods_by_agent`, то есть "получить товары по поставщику".
2. Входные параметры. В рамках данной задачи на вход нам поступает только имя поставщика, поэтому введем входную переменную `agent_name`, тип который мы позаимствуем из таблицы `Agent` у соответствующего атрибута `name_ag`.
3. Локальные переменные. Оставим эту секцию пока пустой.
4. Выходные параметры. Нам нужно вернуть таблицу, которая будет содержать названия товаров и даты последней их поставки. Поэтому выходными параметрами будут переменные `goods_name` и `last_op_date` соответственно.

Код примет следующий вид:
```sql
create or alter procedure get_goods_by_agent(
    agent_name type of column agent.name_ag)
returns (
    goods_name type of column goods.nomenclature,
    last_op_date type of column operation.op_date)
as
begin
  /* Будущий код */
end
```

<a name="написание_логики"></a> 
## Написание логики
<a name="проверка_входных_параметров"></a> 
### 1. Проверка входных параметров
Нам на вход поступает имя поставщика. Необходимо проверить, реально ли есть такой поставщик в таблице `Agent`. И если такового нет, нужно бросить исключение и в автономной транзакции написать в таблицу `log_file` сообщение об ошибке. Поскольку это уже было рассмотрено в [Задаче №3](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/task_3.md), в блоке [2.i. Проверка входных параметров](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/task_3.md#%D0%BF%D1%80%D0%BE%D0%B2%D0%B5%D1%80%D0%BA%D0%B0_%D0%B2%D1%85%D0%BE%D0%B4%D0%BD%D1%8B%D1%85_%D0%BF%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D0%BE%D0%B2), мы опишем логику происходящего кратко. Сначала нам нужно получить id поставщика и сохранить его в какую-нибудь переменную. Для этого объявим локальную переменную `agent_id`:
```sql
declare variable agent_id type of column agent.id_ag;
```
И сделаем запрос на получение id поставщика по его имени:
```sql
:agent_id = (select id_ag from agent where name_ag = :agent_name);
```
Затем, если данный id имеет null-значение, необходимо бросить исключение и сделать запись с сообщением об ошибке в таблицу `log_file`:
```sql
if (:agent_id is null) then
begin
  in autonomous transaction do
      insert into log_file (inform) 
      values ('В базе данных нет поставщика '||:agent_name);
  exception error ;
end
```
На этом проверка закончена. Код к текущему моменту будет иметь вид:
```sql
create or alter procedure get_goods_by_agent(
    agent_name type of column agent.name_ag)
returns (
    goods_name type of column goods.nomenclature,
    last_op_date type of column operation.op_date)
as
declare variable agent_id type of column agent.id_ag;
begin
  :agent_id = (select id_ag from agent where name_ag = :agent_name);
  if (:agent_id is null) then
  begin
    in autonomous transaction do
        insert into log_file (inform) 
        values ('В базе данных нет поставщика '||:agent_name);
    exception error ;
  end
end
```

<a name="решение_основной_задачи"></a> 
### 2. Решение основной задачи
Решение задачи можно разбить на два шага.

1. Сформируем SQL-запрос. Нам необходимо сделать запрос, который вернет для данного поставщика товары, поставленные с помощью операции R, а также даты последних таких операций. Для этого оставим только операции, связанные с нашим поставщиком и имеющие тип R:
```sql
select *
from operation o
where o.id_ag = :agent_id and o.typeop = 'R'
```
Затем поскольку нам нужно группировать по товарам и выводить их название, выполним соединение с таблицей `Goods`:
```sql
select *
from operation o join goods g using (id_goods)
where o.id_ag = :agent_id and o.typeop = 'R'
```
И выполним группировку по атрибуту `nomenclature`:
```sql
select g.nomenclature
from operation o join goods g using (id_goods)
where o.id_ag = :agent_id and o.typeop = 'R'
group by g.nomenclature
```
Наконец, добавим в select вывод даты самой поздней операции в группе для каждого товара - сделать это можно, просто взяв максимум по датам:
```sql
select g.nomenclature, max(o.op_date) as last_date
from operation o join goods g using (id_goods)
where o.id_ag = :agent_id and o.typeop = 'R'
group by g.nomenclature
```
Запрос готов.

2. Поместим данный SQL-запрос под явный курсор. Для этого нужно сделать следующую запись в области локальных переменных:
```sql
declare op_cursor cursor for (select g.nomenclature, max(o.op_date) as last_date
                              from operation o join goods g using (id_goods)
                              where o.id_ag = :agent_id and o.typeop = 'R'
                              group by g.nomenclature);
```
Тогда код будет иметь вид:
```sql
create or alter procedure get_goods_by_agent(
    agent_name type of column agent.name_ag)
returns (
    goods_name type of column goods.nomenclature,
    last_op_date type of column operation.op_date)
as
declare variable agent_id type of column agent.id_ag;
declare op_cursor cursor for (select g.nomenclature, max(o.op_date) as last_date
                              from operation o join goods g using (id_goods)
                              where o.id_ag = :agent_id and o.typeop = 'R'
                              group by g.nomenclature);
begin
  :agent_id = (select id_ag from agent where name_ag = :agent_name);
  if (:agent_id is null) then
  begin
    in autonomous transaction do
        insert into log_file (inform) 
        values ('В базе данных нет поставщика '||:agent_name);
    exception error ;
  end
end
```

<a name="вывод_результирующей_таблицы"></a> 
### 3. Вывод результирующей таблицы
В секции begin-end после проверки входных параметров нужно пройти с помощью курсора по сформированной таблице и заполнить выходные параметры. Код практически полностью совпадает с тем описанием, которые было приведено в [Теории](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab10/readme.md#%D1%82%D0%B5%D0%BE%D1%80%D0%B8%D1%8F), и имеет следующий вид:
```sql
open op_cursor;
while (true) do
begin
  fetch op_cursor into :goods_name, :last_op_date;
  if (row_count = 0) then
      leave;
  suspend;
end
close op_cursor;
```
То есть мы открываем курсор, заходим в бесконечный цикл, с помощью оператора `fetch` записываем в выходные переменные атрибуты текущей записи сформированной таблицы, и если на последнем вызове оператора `fetch` читать было нечего , то выходим, а если было что, то возвращаем значения выходных переменных во внешний контекст с помощью вызова оператора `suspend`. По завершении цикла закрываем курсор. Итоговая процедура будет иметь вид, описанный ниже.

<a name="итоговый_код_процедуры"></a> 
## Итоговый код процедуры
```sql
create or alter procedure get_goods_by_agent(
    agent_name type of column agent.name_ag)
returns (
    goods_name type of column goods.nomenclature,
    last_op_date type of column operation.op_date)
as
declare variable agent_id type of column agent.id_ag;
declare op_cursor cursor for (select g.nomenclature, max(o.op_date) as last_date
                              from operation o join goods g using (id_goods)
                              where o.id_ag = :agent_id and o.typeop = 'R'
                              group by g.nomenclature);
begin
  :agent_id = (select id_ag from agent where name_ag = :agent_name);
  if (:agent_id is null) then
  begin
    in autonomous transaction do
        insert into log_file (inform) 
        values ('В базе данных нет поставщика '||:agent_name);
    exception error ;
  end

  open op_cursor;
  while (true) do
  begin
    fetch op_cursor into :goods_name, :last_op_date;
    if (row_count = 0) then
      leave;
    suspend;
  end
  close op_cursor;
end
```
