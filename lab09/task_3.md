# Задача №3

Нам дали задачу со следующим условием:

**По названию склада и периоду выдать список товаров и их 
оборот за указанный период.**

Рассмотрим решение данной задачи подробно и поэтапно, в соответствии со структурой, описанной в документе ["Как писать процедуры?"](https://github.com/NikitaBogoslovskiy/DatabaseCourse/tree/main/lab09/how-to-create-procedures.md) - то есть сначала настройка процедуры, а затем написание основной логики.

Навигация по решению:

[Настройка процедуры](#настройка_процедуры)
1. [Дадим имя процедуре](#дадим_имя_процедуре).
2. [Настроим входные аргументы процедуры](#настроим_входные_аргументы_процедуры)
3. [Настроим локальные переменные](#настроим_локальные_переменные)
4. [Настроим выходные параметры](#настроим_выходные_параметры)

[Написание логики](#написание_логики)

1. [Проверка входных параметров](#проверка_входных_параметров)
2. [Решение основной задачи](#решение_основной_задачи)
3. [Вывод результирующей таблицы](#вывод_результирующей_таблицы)

[Итоговый код процедуры](#итоговый_код_процедуры)

<a name="настройка_процедуры"></a>
## Настройка процедуры
<a name="дадим_имя_процедуре"></a>
### 1. Дадим процедуре имя
Например, `get_goods_stats_by_warehouse`:
```sql
create or alter procedure get_goods_stats_by_warehouse
```
Если переводить, то процедура называется "получить статистику товаров по складу". Задавать название можно и с помощью графического интерфейса, как это было проиллюстрировано в ["Как писать процедуры?"](https://github.com/NikitaBogoslovskiy/DatabaseCourse/tree/main/lab09/how-to-create-procedures.md), но здесь будет приводиться решение с помощью обычного кода.

<a name="настроим_входные_аргументы_процедуры"></a>
### 2. Настроим входные аргументы процедуры
По условию видно, что нам на вход подают название склада, а также период времени. Период можно представить в виде двух дат - начальной и конечной. Назовем переменную, обозначающую название склада, `warehouse_name`, а начальную и конечную даты - `start_date` и `end_date` соответственно. Поскольку мы будем работать с таблицами `Operation` и `Agent`, типы можно позаимствовать из них с помощью конструкции `type of column`.
```sql
create or alter procedure get_goods_stats_by_warehouse (
    warehouse_name type of column warehouse.name,
    start_date type of column operation.op_date,
    end_date type of column operation.op_date)
```

<a name="настроим_локальные_переменные"></a>
### 3. Настроим локальные переменные
Они нам могут понадобиться для хранения промежуточных значений. Не всегда легко определить, какие переменные нам пригодятся в процессе написания кода, поэтому эту секцию можно обновлять позже по мере необходимости. Оставим пока ее пустой.

<a name="настроим_выходные_параметры"></a>
### 4. Настроим выходные параметры
В задаче нам сказано, что мы должны вернуть таблицу (список), которая будет содержать товары и их оборот. Товары можно представлять в виде id и названия, оборот же представляется в виде двух колонок - A и R. Колонка A будет содержать суммарное количество привезенного товара, колонка R - суммарное количество увезенного товара. В выходные параметры мы записываем столбцы будущей результирующей таблицы, поэтому запишем в выходные параметры `goods_id` (id товара), `goods_name` (название товара), `a_quantity` (количество привезенного товар), `r_quantity` (количество увезенного товара). То есть итоговая таблица, которая вернется из процедуры и будет отвечать на поставленный вопрос в задаче, будет содержать четыре столбца с этими названиями. Запишем это все ниже входных аргументов в секции `returns`:
```sql
create or alter procedure get_goods_stats_by_warehouse (
    warehouse_name type of column warehouse.name,
    start_date type of column operation.op_date,
    end_date type of column operation.op_date)
returns (
    goods_id type of column goods.id_goods,
    goods_name type of column goods.nomenclature,
    a_quantity type of column operation.quantity,
    r_quantity type of column operation.quantity)
```
Можно видеть, что типы переменных мы снова позаимствовали из таблиц `Goods` и `Operation`, поскольку id и название товара - это уже знакомые нам столбцы `id_goods` и `nomenclature`, а суммарные количества товара имеют тот же тип, что и просто количество товара - `quantity`.

<a name="написание_логики"></a>
## Написание логики
Логика процедуры будет идти в секции begin-end после объявления входных, локальных и выходных параметров:
```sql
create or alter procedure get_goods_stats_by_warehouse (
    warehouse_name type of column warehouse.name,
    start_date type of column operation.op_date,
    end_date type of column operation.op_date)
returns (
    goods_id type of column goods.id_goods,
    goods_name type of column goods.nomenclature,
    a_quantity type of column operation.quantity,
    r_quantity type of column operation.quantity)
as
begin
    /* Ваш код */
end
```

<a name="проверка_входных_параметров"></a>
### 1. Проверка входных параметров
Нам дали на вход имя поставщика. А есть ли этот поставщик в таблице `Agent`? Вдруг пользователь ввел какое-то несуществующее название или же просто ошибся в имени поставщика. Для этого нужно выполнить проверку. Самый очевидный способ - попробовать достать id поставщика по переданному нам имени и посмотреть, является он null или нет. Если такой поставщик реально есть, то id будет иметь какое-то не-null значение (типа `p3`). Если же такого поставщика в таблице нет, то id будет просто null. 
Запрос будет иметь вид:
```sql
select id_wh 
from warehouse 
where name = :warehouse_name
```
Можно видеть, что при поиске по имени мы использовали входной аргумент - `warehouse_name`. В рамках запроса, когда мы обращаемся к переменным, следует использовать символ двоеточия - `:warehouse_name`.

Но куда нам сохранить полученный в результате запроса id? Да, запрос-то мы написали, но нам же еще нужно проверить, null или не-null значение имеет полученный id. Именно для этого нужны локальные переменные. Объявим локальную переменную `warehouse_id` после входных и выходных параметров, но до секции begin-end:
```sql
create or alter procedure get_goods_stats_by_warehouse (
    warehouse_name type of column warehouse.name,
    start_date type of column operation.op_date,
    end_date type of column operation.op_date)
returns (
    goods_id type of column goods.id_goods,
    goods_name type of column goods.nomenclature,
    a_quantity type of column operation.quantity,
    r_quantity type of column operation.quantity)
as
declare variable warehouse_id type of column warehouse.id_wh;
begin
    /* Ваш код */
end
```
Можно видеть, что тип у переменной точно такой же, как и у атрибута `id_wh` таблицы `Warehouse`.

Теперь мы можем сохранить результат запроса в переменную:
```sql
:warehouse_id = (select id_wh from warehouse where name = :warehouse_name);
```
Не, ну классно же. Прям язык программирования уже. Теперь нам надо выполнить проверку на null-значение. Сделать это можно с помощью оператора `if ... then ...`:
```sql
if (:warehouse_id is null) then
begin
  /* Код обработки ситуации, когда не найден поставщик с таким именем */
end
```
Как будем обрабатывать такую ситуацию? Во-первых, надо бросить исключение, чтобы автор неправильного имени поставщика увидел явно, что он не прав. Во-вторых, в таблицу `log_file` надо занести какое-то описание возникшего исключения, что пользователь понял, почему он не прав. Кажется, что мы можем написать такой код:
```sql
if (:warehouse_id is null) then
begin
  insert into log_file (inform) 
  values ('В базе данных нет склада с именем '||:warehouse_name);
  exception error;
end
```
Он пишет в таблицу `log_file` в столбец `inform` сообщение об ошибке и бросает исключение с помощью команды `exception error`. Но в этом коде есть одна небольшая проблема: когда мы бросаем исключение, все изменения, сделанные раннее - в том числе добавление сообщения об ошибке - отменяются. То есть наше сообщение не дойдет до таблицы `log_file`. Избежать это довольно просто - нам нужно выполнить запись сообщения в другой автономной транзакции. Для этого достаточно написать `in autonomous transaction do ...`:

```sql
if (:warehouse_id is null) then
begin
  in autonomous transaction do
      insert into log_file (inform) 
      values ('В базе данных нет склада с именем '||:warehouse_name);
  exception error;
end
```
Вуаля! Мы имеем код, который проверяет id склада на null-значение и в случае такового бросает исключение, записывая сообщение об ошибке с помощью отдельной транзакции. Проверка имени поставщика выполнена.

Среди входных параметров у нас есть еще начальная и конечная даты - теоретически их тоже можно проверить, например, чтобы начальная дата была меньше конечной. Но это является необязательным, поскольку если конечная дата будет меньше начальной, в результирующую таблицу просто не попадут записи и она будет пустой - поведение вполне логичное.

На текущий момент код имеет вид:
```sql
create or alter procedure get_goods_stats_by_warehouse (
    warehouse_name type of column warehouse.name,
    start_date type of column operation.op_date,
    end_date type of column operation.op_date)
returns (
    goods_id type of column goods.id_goods,
    goods_name type of column goods.nomenclature,
    a_quantity type of column operation.quantity,
    r_quantity type of column operation.quantity)
as
declare variable warehouse_id type of column warehouse.id_wh;
begin
  :warehouse_id = (select id_wh from warehouse where name = :warehouse_name);
  if (:warehouse_id is null) then
  begin
    in autonomous transaction do
        insert into log_file (inform) 
        values ('В базе данных нет склада с именем '||:warehouse_name);
    exception error ;
  end
end
```

<a name="решение_основной_задачи"></a>
### 2. Решение основной задачи
Эх, пока дошли до решения задачи, саму задачу уже и забыли. Она звучала так:

**По названию склада и периоду выдать список товаров и их 
оборот за указанный период.**

Как будем действовать?
1. Обратимся к таблице `Operation` и оставим в ней только операции, которые относятся к складу с id `:warehouse_id` и имеют дату в диапазоне от `:start_date` до `:end_date` (в операторе select оставим пока только звездочку):
```sql
select *
from operation o
where o.id_wh = :warehouse_id and o.op_date between :start_date and :end_date
```
2. Затем сгруппируем все операции по товарам, которые участвовали в данных операциях:
```sql
select *
from operation o
where o.id_wh = :warehouse_id and o.op_date between :start_date and :end_date
group by o.id_goods
```
3. Наконец-то обратимся к select и укажем атрибуты, которые мы хотели видеть в результирующей таблице - id товара, название товара, количество привезенного товара, количество увезенного товара. Все, кроме названия товара, есть в таблице `Operation`, название же хранится в таблице `Goods`. Выполним с ней соединение:
```sql
select *
from operation o join goods g using (id_goods)
where o.id_wh = :warehouse_id and o.op_date between :start_date and :end_date
group by o.id_goods
```
Не забываем, что у нас внизу висит группировка - это значит, что в операторе select могут быть только либо атрибуты, по которым происходила группировка, либо выражения с использованием агрегирующих функций типа count, sum и других. Поскольку мы хотим помимо `id_goods` вывести еще и название товара `nomenclature`, надо добавить его в секцию группировки и затем со спокойной совестью включить в select вместе с `id_goods`:
```sql
select o.id_goods, g.nomenclature
from operation o join goods g using (id_goods)
where o.id_wh = :warehouse_id and o.op_date between :start_date and :end_date
group by o.id_goods, g.nomenclature
```
Осталось вывести оборот товара. Сейчас мы имеем для каждого товара группу операций, связанных с этим товаром - там могут быть как операции привоза, так и операции увоза. Как посчитать, например, только операции типа A, то есть привоза? Попробуем сделать следующее:
```sql
sum(iif(o.typeop = 'A', o.quantity, 0))
```
Если в группе операций по данному товару встречается операция типа A, мы берем количество привезенного товара с помощью функции `iif` и добавляем его в общую сумму с помощью функции суммирования `sum`. Если же операция имеет тип, отличный от A, то тогда мы добавляем в общую сумму 0, то есть не учитываем данную операцию.

Аналогично мы можем получить количество увезенного товара:
```sql
sum(iif(o.typeop = 'R', o.quantity, 0))
```
Внесем написанные выражения в select и получим искомый запрос:
```sql
select g.id_goods, 
       g.nomenclature, 
       sum(iif(o.typeop = 'A', o.quantity, 0)), 
       sum(iif(o.typeop = 'R', o.quantity, 0))
from operation o join goods g using(id_goods)
where o.id_wh = :warehouse_id and o.op_date between :start_date and :end_date
group by g.id_goods, g.nomenclature
```
Данный запрос формирует таблицу, которая для заданного склада содержит перечень товаров и оборот каждого из них в рамках некоторого временного промежутка. Если бы мы были в первом модуле, то задача уже была бы выполнена. Но здесь нам нужно этот результат еще записать в выходные переменные.

<a name="вывод_результирующей_таблицы"></a>
### 3. Вывод результирующей таблицы
На текущий момент у нас есть следующая процедура:
```sql
create or alter procedure get_goods_stats_by_warehouse (
    warehouse_name type of column warehouse.name,
    start_date type of column operation.op_date,
    end_date type of column operation.op_date)
returns (
    goods_id type of column goods.id_goods,
    goods_name type of column goods.nomenclature,
    a_quantity type of column operation.quantity,
    r_quantity type of column operation.quantity)
as
declare variable warehouse_id type of column warehouse.id_wh;
begin
  :warehouse_id = (select id_wh from warehouse where name = :warehouse_name);
  if (:warehouse_id is null) then
  begin
    in autonomous transaction do
        insert into log_file (inform) 
        values ('В базе данных нет склада с именем '||:warehouse_name);
    exception error ;
  end
end
```
И также подготовлен следующий запрос:
```sql
select g.id_goods, 
       g.nomenclature, 
       sum(iif(o.typeop = 'A', o.quantity, 0)), 
       sum(iif(o.typeop = 'R', o.quantity, 0))
from operation o join goods g using(id_goods)
where o.id_wh = :warehouse_id and o.op_date between :start_date and :end_date
group by g.id_goods, g.nomenclature
```
Как вывести таблицу из функции, то есть записать ее в выходные переменные `goods_id`, `goods_name`, `a_quantity` и `r_quantity`? Для этого достаточно перебрать все записи итоговой таблицы (то есть четверки с id, именем, приходом и уходом) с помощью цикла for, поочередно записывая каждую запись в выходные параметры. В общем виде такой цикл имеет вид:
```sql
for <таблица> 
into <выходной_параметр_1>, 
     <выходной_параметр_2>, 
            ...    
     <выходной_параметр_n> 
do suspend;
```
То есть мы берем каждую строчку таблицы, записываем ее в выходные параметры и с помощью команды `suspend` говорим, чтобы эти параметры забрали. Каждую итерацию эти параметры получают новые значения и каждую итерацию эти значения передаются во внешний контекст, откуда была вызвана процедура.

Другими словами, если мы вызовем снаружи нашу процедуру следующим образом:
```sql
select *
from get_goods_stats_by_warehouse('Название склада', 'Дата-1', 'Дата-2')
```
То каждую итерацию значения выходных параметров будут передаваться в select выше и выводиться в результат внешнего запроса, который вызвал процедуру. Как бы построчный возврат результирующей таблицы во внешний контекст. 

Если заменить таблицу на наш запрос из второго пункта и выходные параметры на те, что мы реально указали, мы получим это:
```sql
for select g.id_goods, 
           g.nomenclature, 
           sum(iif(o.typeop = 'A', o.quantity, 0)), 
           sum(iif(o.typeop = 'R', o.quantity, 0))
    from operation o join goods g using(id_goods)
    where o.id_wh = :warehouse_id and o.op_date between :start_date and :end_date
    group by g.id_goods, g.nomenclature
into :goods_id, 
     :goods_name, 
     :a_quantity, 
     :r_quantity 
do suspend;
```
На этом все. Мы написали код вывода полученной таблицы. Итоговый код процедуры приведен ниже.

<a name="итоговый_код_процедуры"></a>
## Итоговый код процедуры
```sql
create or alter procedure get_goods_stats_by_warehouse (
    warehouse_name type of column warehouse.name,
    start_date type of column operation.op_date,
    end_date type of column operation.op_date)
returns (
    goods_id type of column goods.id_goods,
    goods_name type of column goods.nomenclature,
    a_quantity type of column operation.quantity,
    r_quantity type of column operation.quantity)
as
declare variable warehouse_id type of column warehouse.id_wh;
begin
  :warehouse_id = (select id_wh from warehouse where name = :warehouse_name);
  if (:warehouse_id is null) then
  begin
    in autonomous transaction do
        insert into log_file (inform) 
        values ('В базе данных нет склада с именем '||:warehouse_name);
    exception error ;
  end

  for select g.id_goods, 
             g.nomenclature, 
             sum(iif(o.typeop = 'A', o.quantity, 0)), 
             sum(iif(o.typeop = 'R', o.quantity, 0))
      from operation o join goods g using(id_goods)
      where o.id_wh = :warehouse_id and o.op_date between :start_date and :end_date
      group by g.id_goods, g.nomenclature
  into :goods_id, 
       :goods_name, 
       :a_quantity, 
       :r_quantity 
  do suspend;
end
```
