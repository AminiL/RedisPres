# Redis
## История
Название Redis означает Remote Dictionary Server. Проект Redis начался, когда Сальваторе Санфилиппо по прозвищу antirez, первоначальный разработчик Redis, пытался улучшить масштабируемость своего итальянского стартапа, разрабатывая анализатор веб логов в реальном времени. Столкнувшись со значительными проблемами при масштабировании некоторых типов рабочих нагрузок с использованием традиционных систем баз данных, Sanfilippo в 2009 году приступил к созданию прототипа первой пробной версии концепции Redis на Tcl. Позже Санфилиппо перевел этот прототип на язык C и реализовал первый тип данных — список. После нескольких недель успешного использования проекта внутри компании Sanfilippo решил открыть исходный код, объявив о проекте на Hacker News. Проект начал набирать обороты, особенно среди сообщества Ruby, и GitHub и Instagram стали одними из первых компаний, которые его внедрили.

Санфилиппо был принят на работу в VMware в марте 2010 года.

В мае 2013 года Redis спонсировалась Pivotal Software (дочерняя компания VMware).

В июне 2015 года Redis Labs спонсировала разработку.

В октябре 2018 года был выпущен Redis 5.0, представляющий Redis Stream — новую структуру данных, которая позволяет хранить несколько полей и строковых значений с автоматической последовательностью на основе времени для одного ключа.

В июне 2020 года Сальваторе Санфилиппо ушел с поста сопровождающего Redis.

Основные изменения по версиям смотрите на слайде.

## Инструменты
Есть консольные утилиты (CLI) и графические (GUI). Примеры на слайде. Полная информация на [оффициальном сайте](https://redis.io/resources/tools/).

## Engine Database (Движок)
Собственно, тут нужно ответить, почему Redis такой быстрый.

По сути, это однопоточный сервис, которые использует только оперативную память (это быстро). Преимущества однопоточности в том, что нет локов, нет переключения контекстов, всё достаточно просто. Большая часть времени уходит на соединение, поэтому однопоточность - это хороший вариант. Для большей части запросов достаточно хэш-таблицы, но есть и другие встроенные структуры данных. Понятно, что всё это оптимизировано под собственные типы данных. Скорость соедиенения достигается за счёт хитрой комбинации синхронных и асинхронных операций ввода/вывода и комбинации разных интерфейсов уведомлений о событиях (к примеру подключения, записи, чтения): poll, epoll, kqueue. 

## Redis query language

1. Строка (String)
2. Битовый массив (Bitmap)
3. Битовое поле (Bitfield)
4. Хеш-таблица (Hash)
5. Список (List)
6. Множество (Set)
7. Упорядоченное множество (Sorted set)
8. Геопространственные данные (Geospatial)
9. Структура HyperLogLog (HyperLogLog)
10. Поток (Stream)

Redis поддерживает много структур данных, поэтому и различных команд тоже много. [Полный список](https://redis.io/commands/).

Рассмотрим пример скрипта, который добавляет и удаляет ключ - значение в хэштаблицу myhash:
```
alex:~
$ redis-cli 
127.0.0.1:6379> HSET myhash field1 "Hello"
(integer) 1
127.0.0.1:6379>  HGET myhash field1
"Hello"
127.0.0.1:6379> HSET myhash field2 "Hi" field3 "World"
(integer) 2
127.0.0.1:6379> HGET myhash field2
"Hi"
127.0.0.1:6379> HGET myhash field3
"World"
127.0.0.1:6379> HGETALL myhash
1) "field1"
2) "Hello"
3) "field2"
4) "Hi"
5) "field3"
6) "World"
127.0.0.1:6379> FETCHALL
(error) ERR unknown command `FETCHALL`, with args beginning with: 
127.0.0.1:6379> del myhash
(integer) 1
127.0.0.1:6379> HGETALL myhash
(empty array)
127.0.0.1:6379> quit
alex:~
```

## Индексы в Redis 
В Redis из коробки есть только первичные индексы, они же ключи в set/hashset к примеру. Но, к примеру, используя упорядоченные множества, можно сделать вторичные индексы.
[Пример](https://redis.io/docs/manual/patterns/indexes/).

## План выполения запросов
В самом redis нет, но есть в модуле [RediSearch](https://redis.io/docs/stack/search/).
Он же предоставляет вторичные индексы. 

## Транзакции 
Они есть. У них нет rollback-ов. [https://redis.io/docs/manual/transactions/](https://redis.io/docs/manual/transactions/).
```
> MULTI
OK
> INCR foo
QUEUED
> INCR bar
QUEUED
> EXEC
1) (integer) 1
2) (integer) 1
```

## Механизмы восстановления
Есть несколько вариантов:
1. Запись на диск снепшотов по времени. 
2. Запись логов каждой операции. При восстановлении надо заново их всех выполнить.
3. Отключить запись на диск полностью. Используется для кэширования. 
4. Сочетание 1 и 2.

## Шардирование
Есть для Redis Cluster. Не поддерживаются операции с множествами ключей. Тип шардирования: hash partitioning. Простыми словами, каждый шард хранит некоторый набор ключей.

## Data Mining, Data Warehousing и OLAP

Имеет смысл только в качестве кэша для уже существующих решений. 

## Security
Есть авторизация, есть TLS. No sql injections. Хорошее хэширование, не позволяющее взламывать время работы. См. https://redis.io/docs/management/security/

## Кто использует
Twitter

GitHub

Snapchat

Craigslist

StackOverflow