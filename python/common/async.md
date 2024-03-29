### Содержание 
+ [1. Введение](#1-введение)
+ [2. Первое асинхронное приложение](#2-первое-асинхронное-приложение)
+ [3. Асинхронные функции и корутины](#3-асинхронные-функции-и-корутины)
+ [4. Футуры и задачи](#4-футуры-и-задачи)
+ [5. Асинхронные менеджеры контекста и настоящее асинхронное приложение](#5-асинхронные-менеджеры-контекста-и-настоящее-асинхронное-приложение)
+ [Пара слов о лишних сущностях](#пара-слов-о-лишних-сущностях)

## 1. Введение[^](#содержание)
Асинхронное программирование традиционно относят к темам для "продвинутых". Действительно, у новичков часто возникают сложности с практическим освоением асинхронности. В случае python на то есть весьма веские причины:



+ Асинхронность в python была стандартизирована сравнительно недавно. Библиотека asyncio появилась впервые в версии 3.5 (то есть в 2015 году), хотя возможность костыльно писать асинхронные приложения и даже фреймворки, конечно, была и раньше. Соответственно у Лутца она не описана, а, как всем известно, "про что Лутц не написал, того и знать не надо".
+ Рекомендуемый синтаксис асинхронных команд неоднократно менялся уже и после первого появления asyncio. В сети бродит огромное количество статей и роликов, использующих архаичный код различной степени давности, только сбивающий новичков с толку.
+ Официальная документация asyncio (разумеется, исчерпывающая и прекрасно организованная) рассчитана скорее на создателей фреймворков, чем на разработчиков пользовательских приложений. Там столько всего — глаза разбегаются. А между тем: "Вам нужно знать всего около семи функций для использования asyncio" (c) Юрий Селиванов, автор PEP 492, в которой были добавлены инструкции async и await

На самом деле наша повседневная жизнь буквально наполнена асинхронностью.

Утром меня будит будильник в телефоне. Я когда-то давно поставил его на 8:30 и с тех пор он исправно выполняет свою работу. Чтобы понять когда вставать, мне не нужно таращиться на часы всю ночь напролет. Нет нужды и периодически на них посматривать (скажем, с интервалом в 5 минут). Да я вообще не думаю по ночам о времени, мой мозг занят более интересными задачами — просмотром снов, например. Асинхронная функция "подъем" находится в режиме ожидания. Как только произойдет событие "на часах 8:30", она сама даст о себе знать омерзительным Jingle Bells.

Иногда по выходным мы с собакой выезжаем на рыбалку. Добравшись до берега, я снаряжаю и забрасываю несколько донок с колокольчиками. И... Переключаюсь на другие задачи: разговариваю с собакой, любуюсь красотами природы, истребляю на себе комаров. Я не думаю о рыбе. Задачи "поймать рыбу удочкой N" находятся в режиме ожидания. Когда рыба будет готова к общению, одна из удочек сама даст о себе знать звонком колокольчика.

Будь я автором самого толстого в мире учебника по python, я бы рассказывал читателям про асинхронное программирование уже с первых страниц. Вот только написали "Hello, world!" и тут же приступили к созданию "Hello, asynchronous world!". А уже потом циклы, условия и все такое.

Но при написании этой статьи я все же предполагал, что ее читатель уже знаком с основами python и ему не придется втолковывать что такое генераторы или менеджеры контекста.

### Пара слов о терминологии
В настоящем руководстве я стараюсь придерживаться не академических, а сленговых терминов, принятых в русскоязычных командах, в которых мне довелось работать. То есть "корутина", а не "сопрограмма", "футура", а не "фьючерс" и т. д. При всем при том, я еще не столь низко пал, чтобы, скажем, задачу именовать "таской". Если в вашем проекте приняты другие названия, прошу отнестись с пониманием и не устраивать терминологический холивар.

## 2. Первое асинхронное приложение[^](#содержание)
Предположим, у нас есть две функции в каждой из которых есть некая "быстрая" операция (например, арифметическое вычисление) и "медленная" операция ввода/вывода. Детали реализации медленной операции нам сейчас не важны. Будем моделировать ее функцией `time.sleep()`. Наша задача - выполнить обе задачи как можно быстрее.

Традиционное решение "в лоб":
#### Пример 2.1
```python
import time


def fun1(x):
    print(x**2)
    time.sleep(3)
    print('fun1 завершена')


def fun2(x):
    print(x**0.5)
    time.sleep(3)
    print('fun2 завершена')


def main():
    fun1(4)
    fun2(4)


print(time.strftime('%X'))

main()

print(time.strftime('%X'))
```

Никаких сюрпризов - `fun2` честно ждет пока полностью отработает `fun1` (и быстрая ее часть, и медленная) и только потом начинает выполнять свою работу. Весь процесс занимает 3 + 3 = 6 секунд. Строго говоря, чуть больше чем 6 за счет "быстрых" арифметических операций, но в выбранном масштабе разницу уловить невозможно.

Теперь попробуем сделать то же самое, но в асинхронном режиме. Пока просто запустите предложенный код, подробно мы его разберем чуть позже.

#### Пример 2.2
````python
import asyncio
import time


async def fun1(x):
    print(x**2)
    await asyncio.sleep(3)
    print('fun1 завершена')


async def fun2(x):
    print(x**0.5)
    await asyncio.sleep(3)
    print('fun2 завершена')


async def main():
    task1 = asyncio.create_task(fun1(4))
    task2 = asyncio.create_task(fun2(4))

    await task1
    await task2


print(time.strftime('%X'))

asyncio.run(main())

print(time.strftime('%X'))
````

Сюрприз! Мгновенно выполнились быстрые части обеих функций и затем через 3 секунды (3, а не 6!) одновременно появились оба текстовых сообщения. Полное ощущение, что функции выполнились параллельно (на самом деле нет).

А можно аналогичным образом добавить еще одну функцию-соню? Пожалуйста — хоть сто! Общее время выполнения программы будет по-прежнему определяться самой "тормознутой" из них. Добро пожаловать в асинхронный мир!
Что изменилось в коде?

1. Перед определениями функций появился префикс `async`. Он говорит интерпретатору, что функция должна выполняться асинхронно.
2. Вместо привычного t`ime.sleep` мы использовали `asyncio.sleep`. Это "неблокирующий sleep". В рамках функции ведет себя так же, как традиционный, но не останавливает интерпретатор в целом.
3. Перед вызовом асинхронных функций появился префикс `await`. Он говорит интерпретатору примерно следующее: *"я тут возможно немного потуплю, но ты меня не жди — пусть выполняется другой код, а когда у меня будет настроение продолжиться, я тебе маякну"*.
4. На базе функций мы при помощи `asyncio.create_task` создали задачи (что это такое разберем позже) и запустили все это при помощи `asyncio.run`

#### Как это работает:
+ выполнилась быстрая часть функции `fun1`
+ `fun1` сказала интерпретатору "иди дальше, я посплю 3 секунды"
+ выполнилась быстрая часть функции `fun2`
+ `fun2` сказала интерпретатору "иди дальше, я посплю 3 секунды"
+ интерпретатору дальше делать нечего, поэтому он ждет пока ему маякнет первая проснувшаяся функция
+ на доли миллисекунды раньше проснулась `fun1` (она ведь и уснула чуть раньше) и отрапортовала нам об успешном завершении
+ то же самое сделала функция `fun2`

Замените "посплю" на "пошлю запрос удаленному сервису и буду ждать ответа" и вы поймете как работает реальное асинхронное приложение.

Возможно в других руководствах вам встретится "старомодный" код типа:

#### Пример 2.3
```python
import asyncio
import time


async def fun1(x):
    print(x**2)
    await asyncio.sleep(3)
    print('fun1 завершена')


async def fun2(x):
    print(x**0.5)
    await asyncio.sleep(3)
    print('fun2 завершена')


print(time.strftime('%X'))

loop = asyncio.get_event_loop()
task1 = loop.create_task(fun1(4))
task2 = loop.create_task(fun2(4))
loop.run_until_complete(asyncio.wait([task1, task2]))

print(time.strftime('%X'))
```

Результат тот же самый, но появилось неявное упоминание какого-то цикла событий (event loop) и используются аж три функции библиотеки asyncio (`asyncio.wait, asyncio.get_event_loop и asyncio.run_until_complete`) вместо одной `asyncio.run` в предыдущем примере. Кроме того, если вы используете python версии 3.10+, в консоль прилетает раздражающее предупреждение `DeprecationWarning: There is no current event loop`, что само по себе наводит на мысль, что мы делаем что-то слегка не так.

Давайте пока руководствоваться Дзен питона: *"Простое лучше, чем сложное"*, а цикл событий сам придет к нам... в свое время.

### Пара слов о "медленных" операциях
Как правило, это все, что связано с вводом выводом: получение результата http-запроса, файловые операции, обращение к базе данных.

Однако следует четко понимать: для эффективного использования с asyncio любой медленный интерфейс должен поддерживать асинхронные функции. Иначе никакого выигрыша в производительности вы не получите. Попробуйте использовать в примере 2.2 `time.sleep` вместо `asyncio.sleep` и вы поймете о чем я.

Что касается http-запросов, то здесь есть великолепная библиотека `aiohttp`, честно реализующая асинхронный доступ к веб-серверу. С файловыми операциями сложнее. В Linux доступ к файловой системе по определению не асинхронный, поэтому, несмотря на наличие удобной библиотеки `aiofiles`, где-то в ее глубине всегда будет иметь место многопоточный "мостик" к низкоуровневым функциям ОС. С доступом к БД примерно то же самое. Вроде бы, последние версии `SQLAlchemy` поддерживают асинхронный доступ, но что-то мне подсказывает, что в основе там все тот же старый добрый Threadpool. С другой стороны, в веб-приложениях львиная доля задержек относится именно к сетевому общению, так что "не вполне асинхронный" доступ к локальным ресурсам обычно не является бутылочным горлышком.

## 3. Асинхронные функции и корутины[^](#содержание)
Теперь давайте немного разберемся с типами. Вернемся к "неасинхронному" примеру 2.1, слегка модифицировав его:
#### Пример 3.1
````python
import time


def fun1(x):
    print(x**2)
    time.sleep(3)
    print('fun1 завершена')


def fun2(x):
    print(x**0.5)
    time.sleep(3)
    print('fun2 завершена')


def main():
    fun1(4)
    fun2(4)


print(type(fun1))

print(type(fun1(4)))
````
Все вполне ожидаемо. Функция имеет тип `<class 'function'>`, а ее результат - `<class 'NoneType'>`

Теперь аналогичным образом исследуем "асинхронный" пример 2.2:
#### Пример 3.2
```python
import asyncio
import time


async def fun1(x):
    print(x**2)
    await asyncio.sleep(3)
    print('fun1 завершена')


async def fun2(x):
    print(x**0.5)
    await asyncio.sleep(3)
    print('fun2 завершена')


async def main():
    task1 = asyncio.create_task(fun1(4))
    task2 = asyncio.create_task(fun2(4))

    await task1
    await task2


print(type(fun1))

print(type(fun1(4)))
```
Уже интереснее! Класс функции не изменился, но благодаря ключевому слову async она теперь возвращает не `<class 'NoneType'>`, а `<class 'coroutine'>`. Ничто превратилось в нечто! На сцену выходит новая сущность - корутина.

Что нам нужно знать о корутине? На начальном этапе немного. Помните как в python устроен генератор? Ну, это то, что функция начинает возвращать, если в нее добавить `yield` вместо `return`. Так вот, корутина — это разновидность генератора.

Корутина дает интерпретатору возможность возобновить базовую функцию, которая была приостановлена в месте размещения ключевого слова await.

И вот тут начинается терминологическая путаница, которая попила немало крови добрых разработчиков на собеседованиях. Сплошь и рядом корутиной называют саму функцию, содержащую `await`. Строго говоря, это неправильно. Корутина — это то, что возвращает функция с `await`. Чувствуете разницу между `f` и `f()`?

С генераторами, кстати, та же самая история. Генератором как-то повелось называть функцию, содержащую `yield`, хотя по правильному-то она "генераторная функция". А генератор — это именно тот объект, который генераторная функция возвращает.

Далее по тексту мы постараемся придерживаться правильной терминологии: асинхронная (или корутинная) функция — это `f`, а корутина — `f()`. Но если вы в разговоре назовете корутиной асинхронную функцию, беды большой не произойдет, вас поймут. *"Не важно, какого цвета кошка, лишь бы она ловила мышей"* (с) тов. Дэн Сяопин

## 4. Футуры и задачи[^](#содержание)
Продолжим исследовать нашу программу из примера 2.2. Помнится, на базе корутин мы там создали какие-то загадочные задачи:

#### Пример 4.1
```python
import asyncio


async def fun1(x):
    print(x**2)
    await asyncio.sleep(3)
    print('fun1 завершена')


async def fun2(x):
    print(x**0.5)
    await asyncio.sleep(3)
    print('fun2 завершена')


async def main():
    task1 = asyncio.create_task(fun1(4))
    task2 = asyncio.create_task(fun2(4))

    print(type(task1))
    print(task1.__class__.__bases__)

    await task1
    await task2


asyncio.run(main())
```
Ага, значит задача (что бы это ни значило) имеет тип `<class '_asyncio.Task'>`. Привет, капитан Очевидность!

А кто ваша мама, ребята? А мама наша — какая-то еще более загадочная футура (`<class '_asyncio.Future'>`).

В `asyncio` все шиворот-навыворот, поэтому сначала выясним что такое футура (которую мы видим впервые в жизни), а потом разберемся с ее дочкой задачей (с которой мы уже имели честь познакомиться в предыдущем разделе).

Футура (если совсем упрощенно) — это оболочка для некой асинхронной сущности, позволяющая выполнять ее "как бы одновременно" с другими асинхронными сущностями, переключаясь от одной сущности к другой в точках, обозначенных ключевым словом `await`

Кроме того футура имеет внутреннюю переменную "результат", которая доступна через `.result()` и устанавливается через `.set_result(value)`. Пока ничего не надо делать с этим знанием, оно пригодится в дальнейшем.

У футуры на самом деле еще много чего есть внутри, но на данном этапе не будем слишком углубляться. Футуры в чистом виде используются в основном разработчиками фреймворков, нам же для разработки приложений приходится иметь дело с их дочками — задачами.

Задача — это частный случай футуры, предназначенный для оборачивания корутины.

### Все трагически усложняется
Вернемся к примеру 2.2 и опишем его логику заново, используя теперь уже знакомые нам термины — корутины и задачи:

+ корутину асинхронной функции `fun1` обернули задачей `task1`
+ корутину асинхронной функции `fun2` обернули задачей `task2`
+ в асинхронной функции `main` обозначили точку переключения к задаче `task1`
+ в асинхронной функции `main` обозначили точку переключения к задаче `task2`
+ корутину асинхронной функции `main` передали в функцию `asyncio.run`

Бр-р-р, ужас какой... Воистину: *"Во многой мудрости много печали; и кто умножает познания, умножает скорбь"* (Еккл. 1:18)

### Все счастливо упрощается
А можно проще? Ведь понятие корутина нам необходимо, только чтобы отличать функцию от результата ее выполнения. Давайте попробуем временно забыть про них. Попробуем также перефразировать неуклюжие "точки переключения" и вот эти вот все "обернули-передали". Кроме того, поскольку `asyncio.run` — это единственная рекомендованная точка входа в приложение для python 3.8+, ее отдельное упоминание тоже совершенно излишне для понимания логики нашего приложения.

А теперь (барабанная дробь)... Мы вообще уберем из кода все упоминания об асинхронности. Я понимаю, что работать не будет, но все же давайте посмотрим что получится:

#### Пример 4.2 (не работающий)
```python
def fun1(x):
    print(x**2)
    
    # запустили ожидание
    sleep(3)
    
    print('fun1 завершена')


def fun2(x):
    print(x**0.5)
    
    # запустили ожидание
    sleep(3)
    
    print('fun2 завершена')


def main():
    # создали конкурентную задачу из функции fun1
    task1 = create_task(fun1(4))
    
    # создали конкурентную задачу из функции fun2
    task2 = create_task(fun2(4))

    # запустили задачу task1 
    task1
    
    # запустили task2
    task2


main()
```

Кощунство, скажете вы? Нет, я всего лишь честно выполняю рекомендацию великого и ужасного Гвидо ван Россума:

*"Прищурьтесь и притворитесь, что ключевых слова async и await нет"*

Звучит почти как: *"Наденьте зеленые очки и притворитесь, что стекляшки — это изумруды"*

Итак, в "прищуренной вселенной Гвидо":

Задачи — это "ракеты-носители" для конкурентного запуска "боеголовок"-функций.

### А если вообще без задач?
Как это? Ну вот так, ни в какие задачи ничего не заворачивать, а просто эвейтнуть в main() сами корутины. А что, имеем право!

Пробуем:

#### Пример 4.3 (неудачный)

```python
import asyncio
import time


async def fun1(x):
    print(x**2)
    await asyncio.sleep(3)
    print('fun1 завершена')


async def fun2(x):
    print(x**0.5)
    await asyncio.sleep(3)
    print('fun2 завершена')


async def main():
    await fun1(4)
    await fun2(4)


print(time.strftime('%X'))

asyncio.run(main())

print(time.strftime('%X'))
```

Грусть-печаль... Снова 6 секунд как в давнем примере 1.1, ни разу не асинхронном. Боеголовка без ракеты ожидаемо не взлетела.

#### Вывод:
В `asyncio.run` нужно передавать асинхронную функцию с эвейтами на задачи, а не на корутины. Иначе не взлетит. То есть работать-то будет, но сугубо последовательно, без всякой конкурентности.

### Пара слов о конкурентности
С точки зрения разработчика и (особенно) пользователя конкурентное выполнение в асинхронных и многопоточных приложениях выглядит почти как параллельное. На самом деле никакого параллельного выполнения чего бы то ни было в питоне нет и быть не может. Кто не верит — погулите аббревиатуру GIL. Именно поэтому мы используем осторожное выражение "конкурентное выполнение задач" вместо "параллельное".

Нет, конечно, если очень хочется настоящего параллелизма, можно запустить несколько интерпретаторов python одновременно (библиотека multiprocessing фактически так и делает). Но без крайней нужды лучше такими вещами не заниматься, ибо издержки чаще всего будут непропорционально велики по сравнению с профитом.

А что есть "крайняя нужда"? Это приложения-числодробилки. В них подавляющая часть времени выполнения расходуется на операции процессора и обращения к памяти. Никакого ленивого ожидания ответа от медленной периферии, только жесткий математический хардкор. В этом случае вас, конечно, не спасет ни изящная асинхронность, ни неуклюжая мультипоточность. К счастью, такие негуманные приложения в практике веб-разработки встречаются нечасто.

## 5. Асинхронные менеджеры контекста и настоящее асинхронное приложение[^](#содержание)
Пришло время написать на `asyncio` не тупой перебор неблокирующих слипов, а что-то выполняющее действительно осмысленную работу. Но прежде чем приступить, разберемся с асинхронными менеджерами контекста.

Если вы умеете работать с обычными менеджерами контекста, то без труда освоите и асинхронные. Тут используется знакомая конструкция `with`, только с префиксом `async`, и те же самые контекстные методы, только с буквой a в начале.
#### Пример 5.1
```python
import asyncio


# имитация  асинхронного соединения с некой периферией
async def get_conn(host, port):
    class Conn:
        async def put_data(self):
            print('Отправка данных...')
            await asyncio.sleep(2)
            print('Данные отправлены.')

        async def get_data(self):
            print('Получение данных...')
            await asyncio.sleep(2)
            print('Данные получены.')

        async def close(self):
            print('Завершение соединения...')
            await asyncio.sleep(2)
            print('Соединение завершено.')

    print('Устанавливаем соединение...')
    await asyncio.sleep(2)
    print('Соединение установлено.')
    return Conn()


class Connection:
    # этот конструктор будет выполнен в заголовке with
    def __init__(self, host, port):
        self.host = host
        self.port = port

    # этот метод будет неявно выполнен при входе в with
    async def __aenter__(self):
        self.conn = await get_conn(self.host, self.port)
        return self.conn

    # этот метод будет неявно выполнен при выходе из with
    async def __aexit__(self, exc_type, exc, tb):
        await self.conn.close()


async def main():
    async with Connection('localhost', 9001) as conn:
        send_task = asyncio.create_task(conn.put_data())
        receive_task = asyncio.create_task(conn.get_data())

        # операции отправки и получения данных выполняем конкурентно
        await send_task
        await receive_task


asyncio.run(main())
```

Создавать свои асинхронные менеджеры контекста разработчику приложений приходится нечасто, а вот использовать готовые из асинхронных библиотек — постоянно. Поэтому нам полезно знать, что находится у них внутри.

Теперь, зная как работают асинхронные менеджеры контекста, можно написать ну очень полезное приложение, которое узнает погоду в разных городах при помощи библиотеки aiohttp и API-сервиса openweathermap.org:

#### Пример 5.2
```python
import asyncio
import time
import aiohttp


async def get_weather(city):
    async with aiohttp.ClientSession() as session:
        url = f'http://api.openweathermap.org/data/2.5/weather' \
              f'?q={city}&APPID=2a4ff86f9aaa70041ec8e82db64abf56'

        async with session.get(url) as response:
            weather_json = await response.json()
            print(f'{city}: {weather_json["weather"][0]["main"]}')


async def main(cities_):
    tasks = []
    for city in cities_:
        tasks.append(asyncio.create_task(get_weather(city)))

    for task in tasks:
        await task


cities = ['Moscow', 'St. Petersburg', 'Rostov-on-Don', 'Kaliningrad', 'Vladivostok',
          'Minsk', 'Beijing', 'Delhi', 'Istanbul', 'Tokyo', 'London', 'New York']

print(time.strftime('%X'))

asyncio.run(main(cities))

print(time.strftime('%X'))
```
Опрос 12-ти городов на моем канале 100Mb занимает доли секунды.

Обратите внимание, мы использовали два вложенных менеджера контекста: для сессии и для функции get. Так требует документация aiohttp, не будем с ней спорить.

Давайте попробуем реализовать тот же самый функционал, используя классическую синхронную библиотеку requests и сравним скорость:

#### Пример 5.3
```python
import time
import requests


def get_weather(city):
    url = f'http://api.openweathermap.org/data/2.5/weather' \
          f'?q={city}&APPID=2a4ff86f9aaa70041ec8e82db64abf56'

    weather_json = requests.get(url).json()
    print(f'{city}: {weather_json["weather"][0]["main"]}')


def main(cities_):
    for city in cities_:
        get_weather(city)


cities = ['Moscow', 'St. Petersburg', 'Rostov-on-Don', 'Kaliningrad', 'Vladivostok',
          'Minsk', 'Beijing', 'Delhi', 'Istanbul', 'Tokyo', 'London', 'New York']

print(time.strftime('%X'))

main(cities)

print(time.strftime('%X'))
```

Работает превосходно, но... В среднем занимает 2-3 секунды, то есть раз в 10 больше чем в асинхронном примере. Что и требовалось доказать.

А может ли асинхронная функция не просто что-то делать внутри себя (например, запрашивать и выводить в консоль погоду), но и возвращать результат? Ту же погоду, например, чтобы дальнейшей обработкой занималась функция верхнего уровня main().

Нет ничего проще. Только в этом случае для группового запуска задач необходимо использовать уже не цикл с `await`, а функцию `asyncio.gather`

Давайте попробуем:
#### Пример 5.4
```python
import asyncio
import time
import aiohttp


async def get_weather(city):
    async with aiohttp.ClientSession() as session:
        url = f'http://api.openweathermap.org/data/2.5/weather' \
              f'?q={city}&APPID=2a4ff86f9aaa70041ec8e82db64abf56'

        async with session.get(url) as response:
            weather_json = await response.json()
            return f'{city}: {weather_json["weather"][0]["main"]}'


async def main(cities_):
    tasks = []
    for city in cities_:
        tasks.append(asyncio.create_task(get_weather(city)))

    results = await asyncio.gather(*tasks)

    for result in results:
        print(result)


cities = ['Moscow', 'St. Petersburg', 'Rostov-on-Don', 'Kaliningrad', 'Vladivostok',
          'Minsk', 'Beijing', 'Delhi', 'Istanbul', 'Tokyo', 'London', 'New York']

print(time.strftime('%X'))

asyncio.run(main(cities))

print(time.strftime('%X'))
```

Красиво получилось! Обратите внимание, мы использовали выражение со звездочкой `*tasks` для распаковки списка задач в аргументы функции `asyncio.gather`.

## Пара слов о лишних сущностях[^](#содержание)
Кажется, я совершил невозможное. Настучал уже почти тысячу строк текста и ни разу не упомянул о цикле событий. Ну, почти ни разу. Один раз все-же упомянул: в примере 2.3 "как не надо делать". А между тем, в традиционных руководствах по asyncio этим самым циклом событий начинают душить несчастного читателя буквально с первой страницы. На самом деле цикл событий в наших программах присутствует, но он надежно скрыт от посторонних глаз высокоуровневыми конструкциями. До сих пор у нас не возникало в нем нужды, вот и я и не стал плодить лишних сущностей, руководствуясь принципом дорогого товарища Оккама.

