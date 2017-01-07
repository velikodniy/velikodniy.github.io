---
layout: post
title:  Управление мощной нагрузкой
---

Виды управления
===============

На практике часто возникает необходимость управлять при помощи
цифровой схемы (например, микроконтроллера) каким-то мощным
электрическим прибором. Это может быть мощный светодиод, потребляющий
большой ток, или прибор, питающийся от сети 220 В.

Будем считать, что нам нужно только включать или выключать нагрузку с
низкой частотой. Части схем, решающие эту задачу, называют
ключами. ШИМ-регуляторы, диммеры и прочее рассматривать не будем
(почти).

Условно можно выделить 3 группы методов:

1. Управление нагрузкой постоянного тока.
    - Транзисторный ключ на биполярном транзисторе.
    - Транзисторный ключ на МОП-транзисторе (MOSFET).
    - Транзисторный ключ на IGBT.
2. Управление нагрузкой переменного тока.
    - Тиристорный ключ.
    - Симисторный ключ.
3. Универсальный метод.
    - Реле.

Выбор способа управления зависит как от типа нагрузки, так и от вида
применяемой цифровой логики. Если схема построена на ТТЛ-микросхемах,
то следует помнить, что они управляются током, в отличие от КМОП, где
управление осуществляется напряжением. Иногда это важно.

Ключ на биполярном транзисторе
==============================

Простейший ключ
---------------

Простейший ключ на [биполярном
транзисторе](https://geektimes.ru/post/253730/) проводимости n-p-n
выглядит следующим образом.

![Простейший ключ]({{ site.url }}/assets/2017-01-06-power-control/bjt-simple.png)

Вход слева подключается к цифровой схеме. Если у нас цифровая схема
построена на основе КМОП-логики с [двухтактным («push-pull»)
выходом](https://ru.wikipedia.org/wiki/%D0%94%D0%B2%D1%83%D1%85%D1%82%D0%B0%D0%BA%D1%82%D0%BD%D1%8B%D0%B9_%D0%B2%D1%8B%D1%85%D0%BE%D0%B4),
то логическая «1» фактически означает подключение этого входа к
питанию, а логический «0» — к земле.

Таким образом, при подаче «1» на вход нашей схемы ток от источника
питания потечёт через резистор R1, базу и эмиттер на землю. При этом
транзистор откроется (если, конечно, ток достаточно большой), и ток
сможет идти через переход коллектор — эмиттер, а значит и через
нагрузку.

Резистор R1 играет важную роль — он ограничивает ток через переход
база — эмиттер. Если бы его не было, ток не был бы ничем ограничен и
просто испортил бы управляющую микросхему (ведь именно она связывает
линию питания с транзистором).

Максимальный ток через один выход микроконтроллера обычно ограничен
значением около 25 мА (для STM32).  В интернете можно встретить
утверждения, что микроконтроллеры AVR выдерживают ток в 200 мА, но это
относится ко всем выводам в сумме. Предельное допустимое значение тока
на один вывод примерно такое же — 20-40 мА.

Это, кстати, означает, что подключать светодиоды напрямую к выводам
нельзя. Без токоограничивающих резисторов, микросхема просто сгорит, а
с ними светодиодам не будет хватать тока, чтобы светить ярко.

Обратите внимание, что нагрузка (LOAD) подключена к коллектору, то
есть «сверху». Если подключить её «снизу», у нас возникнет несколько
проблем.

Допустим, мы хотим при помощи 5 В (типичное значение для цифровых
схем) управлять нагрузкой в 12 В. Это значит, что на базе мы можем
получить максимум 5 В. А с учётом падения напряжения на переходе база
— эмиттер, на эмиттере будет напряжение ещё меньше. Если падение
напряжения на переходе равно 0,7 В,то получаем, что на нагрузку
остаётся только 4,3 В, чего явно недостаточно. Если это, например,
реле, оно просто не сработает. Напряжение не может быть выше, иначе
тока через базу вообще не будет. Наличие падения напряжения на
нагрузке также приведёт к уменьшению тока через базу.

Для расчёта сопротивления R1 нужно вспомнить соотношение для
упрощённой модели транзистора:

$$I_к = \beta I_б.$$

Коэффициент $\beta$ — это коэффициент усиления по току. Его ещё
обозначают $h_{21э}$ или $h_{FE}$. У разных транзисторов он
разный.

Зная мощность нагрузки $P$ и напряжение питания $V$, можно найти ток
коллектора, а из него и ток базы:

$$I_б = \frac1{\beta} \frac{P}{V}.$$

По закону Ома получаем:

$$R_1 = \frac{V}{I_б}.$$

Коэффициент $\beta$ не фиксированная величина, он может меняться даже
для одного транзистора в зависимости от режима работы, поэтому лучше
брать значение тока базы при расчёте чуть больше, чтобы был запас по
току коллектора. Главное помнить, что ток базы не должен превышать
предельно допустимое для микросхемы.

Также важно при выборе модели транзистора помнить о предельном токе
коллектора и напряжении коллектор — эмиттер.

Ниже как пример приведены характеристики некоторых популярных
транзисторов с проводимостью n-p-n.

| Модель        | $\beta$   | $\max\ I_{к}$ | $\max\ V_{кэ}$   |
| :------------ |:---------:|:-------------:|:----------------:|
| КТ315Г        | 50…350    | 100 мА        | 35 В             |
| КТ3102Е       | 400…1000  | 100 мА        | 50 В             |
| MJE13002      | 25…40     | 1,5 А         | 600 В            |
| 2SC4242       | 10        | 7 А           | 400 В            |

Модели выбраны случайно, просто это транзисторы, которые легко найти
или откуда-то выпаять. Для ключа в рассматриваемой схеме, конечно,
можно использовать любой n-p-n-транзистор, подходящий по параметрам и
цене.

Доработка схемы
---------------

Если вход схемы подключен к push-pull выходу, то особой доработки не
требуется. Рассмотрим случай, когда вход — это просто выключатель,
который либо подтягивает базу к питанию, либо оставляет её «висеть в
воздухе». Тогда для надёжного закрытия транзистора нужно добавить ещё
один резистор, выравнивающий напряжение между базой и эмиттером.

Кроме того, нужно помнить, что если нагрузка индуктивная, то
обязательно нужен защитный диод. Дело в том, что энергия, запасённая
магнитным полем, не даёт мгновенно уменьшить ток до нуля при
отключении ключа. А значит, на контактах нагрузки возникнет напряжение
обратной полярности, которое легко может нарушить работу схемы или
даже повредить её.

Совет касательно защитного диода универсальный и в равной степени
относится и к другим видам ключей.

Если нагрузка резистивная, то диод не нужен.

В итоге усовершенствованная схема принимает следующий вид.

![Усовершенствованная схема на биполярном ключе]({{ site.url }}/assets/2017-01-06-power-control/bjt-advanced.png)

Резистор R2 обычно берут с сопротивлением, в 10 раз большим, чем
сопротивление R1, чтобы образованный этими резисторами делитель не
понижал слишком сильно напряжение между базой и эмиттером.

Для нагрузки в виде реле можно добавить ещё несколько
усовершенствований. Оно обычно кратковременно потребляет большой ток
только в момент переключения, когда тратится энергия на замыкание
контакта. В остальное время ток через него можно (и нужно) ограничить
резистором, так как удержание контакта требует меньше энергии.

Для этого можно применить схему, приведённую ниже.

![Ограничение тока удержания реле]({{ site.url }}/assets/2017-01-06-power-control/bjt-relay.png)

В момент включения реле, пока конденсатор C1 не заряжен, через него
идёт основной ток. Когда конденсатор зарядится (а к этому моменту реле
перейдёт в режим удержания контакта), ток будет идти через резистор
R2. Через него же будет разряжаться конденсатор после отключения реле.

Ёмкость C1 зависит от времени переключения реле. Можно взять,
например, 10 мкФ.

С другой стороны, ёмкость будет ограничивать частоту переключения
реле, хоть и на незначительную для практических целей величину.

Пример расчёта простой схемы
----------------------------

Пусть, например, требуется включать и выключать светодиод с помощью
микроконтроллера. Тогда схема управления будет выглядеть следующим
образом.

![Управление светодиодом]({{ site.url }}/assets/2017-01-06-power-control/bjt-led.png)

Пусть напряжение питания равно 5 В.

Характеристики (рабочий ток и падение напряжения) типичных светодиодов
диаметром 5 мм можно приблизительно оценить по таблице.

| Цвет          | $I_{LED}$ | $V_{LED}$ |
| :-------------|:---------:|:---------:|
| Красный       | 20 мА     | 1,9 В     |
| Зеленый       | 20 мА     | 2,3 В     |
| Желтый        | 20 мА     | 2,1 В     |
| Синий (яркий) | 75 мА     | 3,6 В     |
| Белый (яркий) | 75 мА     | 3,6 В     |

Пусть используется белый светодиод. В качестве транзисторного ключа
используем КТ3102Е — он подходит по максимальному току (100 мА) и
напряжению (50 В). Будем считать, что его коэффициент передачи тока
равен $\beta = 400$.

Итак, если падение напряжения на диоде равно $V_{LED} = 3{,}6\,В$, то
напряжение на резисторе R2 будет равно $V_{R2} = 5{,}0 - 3{,}6 =
1{,}4\,В$. Для рабочего тока светодиода $I_{LED} = 0{,}075\,А$ получаем

$$R_2 = \frac{V_{R2}}{I_{LED}} = \frac{1{,}4}{0{,}075} \approx 20\,Ом.$$

Для тока $I_{LED} = 0{,}075\,А$ управляющий ток должен быть в $\beta =
400$ раз меньше:

$$I_б = \frac{I_{LED}}{\beta} \approx 0{,}2\,мА.$$

Отсюда

$$R_1 = \frac{V}{I_б} \approx 27\,кОм.$$

Но лучше взять значение чуть меньше, чтобы обеспечить запас по току.

Таким образом, мы нашли значения сопротивлений R1 и R2.

Транзистор Дарлингтона
----------------------

Если нагрузка очень мощная, то ток через неё может достигать
нескольких ампер. Для мощных транзисторов коэффициент $\beta$ может
быть недостаточным. (Тем более, как видно из таблицы, для мощных
транзисторов он и так невелик.)

В этом случае можно применять каскад из двух транзисторов. Первый
транзистор управляет током, который открывает второй транзистор. Такая
схема включения называется схемой Дарлингтона.

![Схема Дарлингтона]({{ site.url }}/assets/2017-01-06-power-control/bjt-darl.png)

В этой схеме коэффициенты $\beta$ двух транзисторов умножаются, что
позволяет получить очень большой коэффициент передачи тока.

Для повышения скорости выключения транзисторов можно у каждого соединить
эмиттер и базу резистором.

![Схема Дарлингтона с ускоренным выключением]({{ site.url }}/assets/2017-01-06-power-control/bjt-darl2.png)

Сопротивления должны быть достаточно большими, чтобы не влиять на ток
база — эмиттер. Типичные значения — 5…10 кОм для напряжений 5…12 В.

Выпускаются транзисторы Дарлингтона в виде отдельного прибора. Примеры
таких транзисторов приведены в таблице.

| Модель        | $\beta$   | $\max\ I_{к}$ | $\max\ V_{кэ}$   |
| :------------ |:---------:|:-------------:|:----------------:|
| КТ829В        | 750       | 8 А           | 60 В             |
| BDX54C        | 750       | 8 А           | 100 В            |

В остальном работа ключа остаётся такой же.


Ключ на полевом транзисторе
===========================

Простейший ключ
---------------

В дальнейшем полевым транзистором мы будет называть конкретно MOSFET,
то есть [полевые транзисторы с изолированным
затвором](https://ru.wikipedia.org/wiki/%D0%9C%D0%9E%D0%9F-%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D0%B0)
(они же МОП, они же МДП). Они удобны тем, что управляются
исключительно напряжением: если напряжение на затворе больше
порогового, то транзистор открывается. При этом управляющий ток через
транзистор пока он открыт или закрыт не течёт. Это значительное
преимущество перед биполярными транзисторами, у которых ток течёт всё
время, пока открыт транзистор.

Также в дальнейшем мы будем использовать только n-канальные MOSFET
(даже для двухтактных схем). Это связано с тем, что n-канальные
транзисторы дешевле и имеют лучшие характеристики.

Простейшая схема ключа на MOSFET приведена ниже.

![Простой ключ на MOSFET]({{ site.url }}/assets/2017-01-06-power-control/mosfet-simple.png)

Опять же, нагрузка подключена «сверху», к стоку. Если подключить её
«снизу», то схема не будет работать. Дело в том, что тразистор
открывается, если напряжение между затвором и истоком превышает
пороговое. При подключении «снизу» нагрузка будет давать
дополнительное падение напряжения, и транзистор может не открыться или
открыться не полностью.

Несмотря на то, что MOSFET управляется только напряжением и ток через
затвор не идёт, затвор образует с подложкой паразитный
конденсатор. Когда транзистор открывается или закрывается, этот
конденсатор заряжается или разряжается через вход ключевой схемы. И
если этот вход подключен к push-pull выходу микросхемы, через неё
потечёт довольно большой ток, который может вывести её из строя.

При управлении типа push-pull схема разряда конденсатора образует,
фактически, RC-цепочку, в которой максимальный ток разряда будет равен

$$I_{разр} = \frac{V}{R_1},$$

где $V$ — напряжение, которым управляется транзистор.

Таким образом, достаточно будет поставить резистор на 100 Ом, чтобы
ограничить ток заряда — разряда до 10 мА. Но чем больше сопротивление
резистора, тем медленнее он будет открываться и закрываться, так как
постоянная времени $\tau = RC$ увеличится. Это важно, если транзистор
часто переключается. Например, в ШИМ-регуляторе.

Основные параметры, на которые следует обращать внимание — это
пороговое напряжение $V_{th}$, максимальный ток через сток $I_D$ и
сопротивление сток — исток $R_{DS}$ у открытого транзистора.

Ниже приведена таблица с примерами характеристик МОП-транзисторов.

| Модель        | $V_{th}$  | $\max\ I_D$   | $\max\ R_{DS}$   |
| :------------ |:---------:|:-------------:|:----------------:|
| 2N7000        | 3 В       | 200 мА        | 5 Ом             |
| IRFZ44N       | 4 В       | 35 А          | 0,0175 Ом        |
| IRF630        | 4 В       | 9 А           | 0,4 Ом           |
| IRL2505       | 2 В       | 74 А          | 0,008 Ом         |

Для $V_{th}$ приведены максимальные значения. Дело в том, что у разных
транзисторов даже из одной партии этот параметр может сильно
отличаться. Но если максимальное значение равно, скажем, 3 В, то этот
транзистор гарантированно можно использовать в цифровых схемах с
напряжением питания 3,3 В или 5 В.

Сопротивление сток — исток у приведённых моделей транзисторов
достаточно маленькое, но следует помнить, что при больших напряжениях
управляемой нагрузки даже оно может привести к выделению значительной
мощности в виде тепла.

Схема ускоренного включения
---------------------------

Как уже было сказано, если напряжение на затворе относительно стока
превышает пороговое напряжение, то транзистор открывается и
сопротивление сток — исток мало. Однако, напряжение при включении не
может резко скакнуть до порогового. А при меньших значениях транзистор
работает как сопротивление, рассеивая тепло. Если нагрузку приходится
включать часто (например, в ШИМ-контроллере), то желательно как можно
быстрее переводить транзистор из закрытого состояния в открытое и
обратно.

Относительная медленность переключения транзистора связана опять же с
паразитной ёмкостью затвора. Чтобы паразитный конденсатор зарядился
как можно быстрее, нужно направить в него как можно больший ток. А так
как у микроконтроллера есть ограничение на максимальный ток выходов,
то направить этот ток можно с помощью вспомогательного биполярного
транзистора.

Кроме заряда, паразитный конденсатор нужно ещё и разряжать. Поэтому
оптимальной представляется двухтактная схема на комплементарных
биполярных транзисторах (можно взять, например, КТ3102 и КТ3107).

![Ключ на MOSFET с двухтактным управлением]({{ site.url }}/assets/2017-01-06-power-control/mosfet-pp.png)

Ещё раз обратите внимание на расположение нагрузки для n-канального
транзистора — она расположена «сверху». Если расположить её между
транзистором и землёй, из-за падения напряжения на нагрузке напряжение
затвор — исток может оказаться меньше порогового, транзистор откроется
не полностью и может перегреться и выйти из строя.

Драйвер полевого транзистора
----------------------------

Если всё же требуется подключать нагрузку к n-канальному транзистору
между стоком и землёй, то решение есть. Можно использовать готовую
микросхему — драйвер верхнего плеча. Верхнего — потому что транзистор
сверху.

Выпускаются и драйверы сразу верхнего и нижнего плеч (например,
IR2151) для построения двухтактной схемы, но для простого включения
нагрузки это не требуется. Это нужно, если нагрузку нельзя оставлять
«висеть в воздухе», а требуется обязательно подтягивать к земле.

Рассмотрим схему драйвера верхнего плеча на примере IR2117.

![Драйвер MOSFET]({{ site.url }}/assets/2017-01-06-power-control/mosfet-driver.png)

Схема не сильно сложная, а использование драйвера повзоляет наиболее
эффективно использовать транзистор.

IGBT
----

Ещё один интересный класс полупроводниковых приборов, которые можно
использовать в качестве ключа — это [биполярные транзисторы с
изолированным затвором](https://ru.wikipedia.org/wiki/%D0%91%D0%B8%D0%BF%D0%BE%D0%BB%D1%8F%D1%80%D0%BD%D1%8B%D0%B9_%D1%82%D1%80%D0%B0%D0%BD%D0%B7%D0%B8%D1%81%D1%82%D0%BE%D1%80_%D1%81_%D0%B8%D0%B7%D0%BE%D0%BB%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D1%8B%D0%BC_%D0%B7%D0%B0%D1%82%D0%B2%D0%BE%D1%80%D0%BE%D0%BC) (IGBT).

Они сочетают в себе преимущества как МОП-, так и биполярных
транзисторов: управляются напряжением, имеют большие значения
предельно допустимых напряжений и токов.

Управлять ключом на IGBT можно так же, как и ключом на MOSFET. Из-за
того, что IGBT применяются больше в силовой электронике, они обычно
используются вместе с драйверами.

Например, согласно даташиту, IR2117 можно использовать для управления
IGBT.

![Драйвер IGBT]({{ site.url }}/assets/2017-01-06-power-control/igbt.png)

Пример IGBT — IRG4BC30F. 


Управление нагрузкой переменного тока
=====================================

Все предыдущие схемы отличало то, что нагрузка хоть и была мощной, но
работала от постоянного тока. В схемах была чётко выраженные земля и
линия питания (или две линии — для контроллера и нагрузки).

Для цепей переменного тока нужно использовать другие подходы. Самые
распространённые — это использование тиристоров, симисторов и реле.
Реле рассмотрим чуть позже, а пока поговорим о первых двух.

Тиристоры и симисторы
---------------------

[Тиристор](https://ru.wikipedia.org/wiki/%D0%A2%D0%B8%D1%80%D0%B8%D1%81%D1%82%D0%BE%D1%80)
— это полупроводниковый прибор, который может находится в двух
состояниях:

- открытом — пропускает ток, но только в одном направлении,
- закрытом — не пропускает ток.

Так как тиристор пропускает ток только в одном направлении, для
включения и выключения нагрузки он подходит не очень хорошо.  Половину
времени на каждый период переменного тока прибор простаивает.  Тем не
менее, тиристор можно использовать в диммере. Там он может применяться
для управления мощностью, отсекая от волны питания кусочек требуемой
мощности.

Симистор — это, фактически двунаправленный тиристор. А значит он
позволяет пропускать не полуволны, а полную волну напряжения питания
нагрузки.

Открыть симистор (или тиристор) можно двумя способами:

- подать (хотя бы кратковременно) отпирающий ток на управляющий электрод;
- подать достаточно высокое напряжение на его «рабочие» электроды.

Второй способ нам не подходит, так как напряжение питания у нас будет
постоянной амплитуды.

После того, как симистор открылся, его можно закрыть поменяв
полярность или снизив ток через него то величины, меньшей чем так
называемый ток удержания. Но так как питание организовано переменным
током, это автоматически произойдёт по окончании полупериода.

При выборе симистора важно учесть величину тока удержания
($I_H$). Если взять мощный симистор с большим током удержания, ток
через нагрузку может оказаться слишком маленьким, и симистор просто не
откроется.

Симисторный ключ
----------------

Для гальванической развязки цепей управления и питания лучше
использовать оптопару или специальный симисторный драйвер. Например,
MOC3023M или MOC3052.

Эти оптопары состоят из инфракрасного светодиода и фотосимистора. Этот
фотосимистор можно использовать для управления мощным симисторным
ключом.

В MOC3052 падение напряжения на светодиоде равно 3 В, а ток — 60 мА,
поэтому при подключении к микроконтроллеру, возможно, придётся
использовать дополнительный транзисторный ключ.

Встроенный симистор же рассчитан на напряжение до 600 В и ток до
1 А. Этого достаточно для управления мощными бытовыми приборами через
второй силовой симистор.

Рассмотрим схему управления резистивной нагрузкой (например, лампой
накаливания).

![Управление через симистор]({{ site.url }}/assets/2017-01-06-power-control/triac.png)

Таким образом, эта оптопара выступает в роли драйвера
симистора.

Существуют и драйверы с детектором нуля — например, MOC3061. Они
переключаются только в начале периода, что снижает помехи в
электросети.

Резисторы R1 и R2 рассчитываются как обычно. Сопротивление же
резистора R3 определяется исходя из пикового напряжения в сети питания
и отпирающего тока силового симистора. Если взять слишком большое —
симистор не откроется, слишком маленькое — ток будет течь
напрасно. Резистор может потребоваться мощный.

Нелишним будет напомнить, что 220 В в электросети — это значение
действующего напряжения. Пиковое напряжение равно $\sqrt2 \cdot 220 \approx
310\,В$.

Управление индуктивной нагрузкой
--------------------------------

При управлении индуктивной нагрузкой, такой как электродвигатель, или
при наличии помех в сети напряжение может стать достаточно большим,
чтобы симистор самопроизвольно открылся.  Для борьбы с этим явлением в
схему необходимо добавить снаббер — это сглаживающий конденсатор и
резистор параллельно симистору.

![Управление через симистор со снаббером]({{ site.url }}/assets/2017-01-06-power-control/triac-snubber.png)

Снаббер не сильно улучшает ситуацию с выбросами, но с ним лучше, чем
без него.

Электролитический конденсатор дожен быть рассчитан на напряжение,
большее пикового в сети питания. Ещё раз вспомним, что для 220 В — это
310 В. Лучше брать с запасом.

Есть также модели симисторов, которым не требуется снаббер. Например,
BTA06-600C.

Примеры симисторов
------------------

Примеры симисторов приведены в таблице ниже. Здесь $I_H$ — ток удержания, 
$\max\ I_{T(RMS)}$ — максимальный ток, $\max\ V_{DRM}$ — максимальное напряжение,
$I_{GT}$ — отпирающий ток.

| Модель        | $I_H$        | $\max\ I_{T(RMS)}$ | $\max\ V_{DRM}$  | $I_{GT}$  |
| :------------ |:------------:|:------------------:|:----------------:|:---------:|
| BT134-600D    | 10 мА        | 4 А                | 600 В            | 5 мА      |
| MAC97A8       | 10 мА        | 0,6 А              | 600 В            | 5 мА      |
| Z0607         | 5 мА         | 0,8 А              | 600 В            | 5 мА      |
| BTA06-600C    | 25 мА        | 6 А                | 600 В            | 50 мА     |


Реле
====

Электромагнитные реле
---------------------

С точки зрения микроконтроллера, реле само является мощной нагрузкой,
причём индуктивной. Поэтому для включения или выключения реле нужно
использовать, например, транзисторный ключ. Схема подключения и также
улучшение этой схемы было рассмотрено ранее.

Реле подкупают своей простотой и эффективностью. Например, реле
HLS8-22F-5VDC — управляется напряжением 5 В и способно коммутировать
нагрузку, подтребляющую ток до 15 А.

Твердотельные реле
------------------

Главное преимущество реле — простота использования — омрачается
несколькими недостатками:

- это механический прибор и контакты могу загрязниться или даже привариться друг к другу,
- меньшая скорость переключения,
- сравнительно большие токи для переключения,
- контакты щёлкают.

Часть этих недостатков устранена в так называемых [твердотельных
реле](https://ru.wikipedia.org/wiki/%D0%A2%D0%B2%D0%B5%D1%80%D0%B4%D0%BE%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D0%BE%D0%B5_%D1%80%D0%B5%D0%BB%D0%B5). Это,
фактически, полупроводниковые приборы с гальванической развязкой,
содержащие внутри полноценную схему мощного ключа.

Заключение
==========

Таким образом, в арсенале у нас достаточно способов управления
нагрузкой, чтобы решить практически любую задачу, которая может
возникнуть перед радиолюбителем.

Полезные источники
==================

1. Хоровиц П., Хилл У. Искусство схемотехники. Том 1. — М.: Мир, 1993.
2. [Управление мощной нагрузкой переменного тока](http://easyelectronics.ru/upravlenie-moshhnoj-nagruzkoj-peremennogo-toka.html)
3. [Управление мощной нагрузкой постоянного тока. Часть 1](http://easyelectronics.ru/upravlenie-moshhnoj-nagruzkoj-postoyannogo-toka-chast-1.html)
4. [Управление мощной нагрузкой постоянного тока. Часть 2](http://easyelectronics.ru/upravlenie-moshhnoj-nagruzkoj-postoyannogo-toka-chast-2.html)
5. [Управление мощной нагрузкой постоянного тока. Часть 3](http://easyelectronics.ru/upravlenie-moshhnoj-nagruzkoj-postoyannogo-toka-chast-3.html)
6. [Щелкаем реле правильно: коммутация мощных нагрузок](https://geektimes.ru/company/unwds/blog/271090/)
7. [Управление мощной нагрузкой переменного тока](https://geektimes.ru/post/257416/)
8. [Управление MOSFET-ами #1](https://vasilisks.wordpress.com/2013/01/07/%d1%83%d0%bf%d1%80%d0%b0%d0%b2%d0%bb%d0%b5%d0%bd%d0%b8%d0%b5-mosfet-%d0%b0%d0%bc%d0%b8-1/)
9. [Современные высоковольтные драйверы MOSFET- и IGBT-транзисторов](http://www.compel.ru/lib/ne/2010/6/10-sovremennyie-vyisokovoltnyie-drayveryi-mosfet-i-igbt-tranzistorov)
10. [Ключ на плечо! – особенности применения высоковольтных драйверов производства IR](http://www.compel.ru/lib/ne/2013/5/6-klyuch-na-plecho-osobennosti-primeneniya-vyisokovoltnyih-drayverov-proizvodstva-ir)

Редактор схем
=============

Все схемы нарисованы в [KiCAD](http://kicad-pcb.org/). В последнее
время для своих проектов использую именно его, очень удобно,
рекомендую. С его помощью можно не только чертить схемы, но и
проектировать печатные платы.