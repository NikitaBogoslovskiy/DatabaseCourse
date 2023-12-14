# Как писать процедуры?
В лабораторной работе №8 мы уже писали небольшие процедуры, но на всякий случай еще раз рассмотрим, как создавать процедуру. Процесс этот можно разбить на два этапа - подготовка процедуры и написание основной логики.

## Подготовка процедуры
В каждом пункте будет две картиночки - первая иллюстрирует настройку процедуры с помощью графического интерфейса IBExpert, вторая показывает это в виде обычного кода.
1. Дайте имя процедуре:

![](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/materials/procedure_step1_gui.jpg)
![](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/materials/procedure_step1_code.jpg)

2. Определите входные аргументы процедуры. Например, нас просят вывести статистику по операциям за некоторый промежуток времени, ограниченный двумя датами - значит, в качестве входных аргументов будут две переменные: начальная дата и конечная дата.

![](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/materials/procedure_step2_gui.jpg)
![](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/materials/procedure_step2_code.jpg)

Можно заметить, что тип мы написали не вручную, а позаимствовали из таблицы `Operation`. Так можно писать, если мы хотим получить в точности тот же тип, что и в таблице у некоторого атрибута. Если нам нужен какой-то другой тип - например, целочисленный - то можно написать это вручную:

![](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/materials/procedure_step2_2_gui.jpg)
![](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/materials/procedure_step2_2_code.jpg)

3. Определите локальные переменные. При работе процедуры нам, вероятно, потребуются вспомогательные переменные, в которых мы захотим хранить промежуточные значения. Например, на вход нам поступило имя поставщика, а нам проще работать с его ID - тогда в качестве локальной переменной можно завести переменную `agent_id`. Заводятся они также в отдельном блоке.

![](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/materials/procedure_step3_gui.jpg)
![](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/materials/procedure_step3_code.jpg)

4. Определите выходные аргументы. Это как раз новая часть изучаемого материала. Результатом работы процедуры является таблица. И в блоке выходных аргументов мы определяем атрибуты (столбцы) этой таблицы. Например, в задании нас просят вывести для каждого поставщика число сделанных им операций. Тогда результирующей таблицей будет таблица со столбцами `AGENT_NAME`, `OPERATIONS_NUMBER`. Именно эти два параметра и нужно указать в блоке выходных аргументов. В нем мы определяем, какие столбцы будет иметь итоговая таблица.

![](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/materials/procedure_step4_gui.jpg)
![](https://github.com/NikitaBogoslovskiy/DatabaseCourse/blob/main/lab09/materials/procedure_step4_code.jpg)

Процедура настроена, мы большие молодцы.

## Написание основной логики
Здесь мы не будем подробно останавливаться на том, как и что выглядит, поскольку это лучше разобрать на конкретном примере. Рассмотрим только три основных шага при написании процедуры:
1. Первое, что мы должны сделать - это проверить правильность введенных входных параметров. Например, нам на вход дали имя поставщика. А точно введённый поставщик есть в таблице с поставщиками? Или пользователь ошибся и ввел несуществующее имя? Это и надо проверить.
2. На втором шаге мы решаем поставленную задачу. Как правило, это стандартный и знакомый нам SQL-запрос. Отличие заключается в том, что если мы хотим использовать переменные, к ним лучше обращаться через двоеточие. Например, если у нас есть переменная `agent_id`, в теле процедуры и в SQL-запросе, в частности, лучше писать `:agent_id`.
3. Финальная часть процедуры - вывод полученной информации. Раньше мы записывали результат в таблицу log_file. В этой же лабораторной работе мы будем записывать результат в виде выходной таблицы.