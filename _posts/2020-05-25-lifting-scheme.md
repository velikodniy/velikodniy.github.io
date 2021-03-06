---
layout: post
title: Схема лифтинга
---

У меня уже был [пост]({{ site.url }}/2019/01/31/wavelets-theory/), в котором я попытался кратко изложить взгляд на вейвлет-преобразование с точки зрения сжатия изображений.
Однако та статья совершенно незаслуженно обходит стороной такой интересный подход к сжатию, как схема лифтинга.
Хотя этот подход и связан с вейвлетами, для его понимания уже не нужно прибегать к матрицам.
Всё можно объяснить очень наглядно, что я и попытаюсь сделать в этой небольшой статье.
Я попытался написать её самодостаточной, так что читать предыдущую не обязательно.


Основная идея
=============

Кратко напомню, как именно сжимаются изображения с помощью вейвлетов.
Существует много универсальных алгоритмов сжатия, использующихся в популярных архиваторах, как ZIP, 7zip и других.
Хотя эти алгоритмы могут сильно отличаться друг от друга, идея везде одна — поиск закономерностей в данных.
А самая лучшая для сжатия закономерность — это последовательность одинаковых чисел.

Таким образом, сигнал, в котором много одинаковых элементов (скажем, нулей) подряд, сжать очень легко.
В случае с изображениями (в отличие, например, от текста) можно немного пожертвовать качеством.
Поэтому, если в сигнале элементы будут ненулевые, но очень близкие к нулю, то мы можем просто их обнулить.
Да, при декодировании картинка будет отличаться, но если мы обнулили незначительные детали, то разницу не заметим.

Если мы посмотрим на фотографии из реальной жизни, то заметим, что часто они — это комбинация общего плана и деталей.
Например, портрет.
Общий план — это то, как тени передают форму лица, а детали — это морщинки, неровности кожи.
Где-то больше общего плана (плечо на фото ниже), где-то больше деталей (шляпка).

![Лена Сёдерберг]({{ site.url }}/assets/2020-05-25-lifting-scheme/lena.jpg)

И тут мы можем заметить следующее.
Если бы мы могли разделить сигнал общий план и детали, то на области, где деталей мало, они были бы почти нулевыми.
А если это преобразование ещё и обратимо, то вообще замечательно.

В прошлой статье мы как раз и рассмотрели такой метод разделения, но для последовательности чисел.
Но его можно успешно применить и к изображениям.
Ведь каждая строка пикселей изображения — это последовательность, как и каждый столбец.
Просто применяем метод сперва ко всем строкам, а потом ко всем столбцам.

![Однократное применение преобразования Хаара]({{ site.url }}/assets/2020-05-25-lifting-scheme/lena-haar-1.jpg)

Каждое разделение не увеличивает число пикселей.
Оно заполняет одну половину низкочастотным общим планом, а вторую — высокочастотными деталями.
После применения к строкам получаем две половинка.
Затем после применения к столбцам — четыре фрагмента.
При этом детали занимают $$\frac34$$ пикселей.
Видно, что в областях с деталями очень много чёрного цвета — это и есть нули.

Ничто не мешает применить это же преобразование в верхнему правому углу.
А потом ещё раз, и ещё.
Результат будет содержать очень много околонудевых значений, а значит его можно будет легко сжать.

![Двукратное применение преобразования Хаара]({{ site.url }}/assets/2020-05-25-lifting-scheme/lena-haar-2.jpg)

![Многократное применение преобразования Хаара]({{ site.url }}/assets/2020-05-25-lifting-scheme/lena-haar.jpg)

Попробуем теперь разобраться с самим преобразованием.


Разделение сигнала
==================

Идея разделения сигнала на две части — низкочастотную и высокочастотную — очень привлекательна сама по себе.
Мы можем многократно извлекать детали из всё уменьшающейся низкочастотной картинки, а потом просто отбросить все детали, которые слишком малы, чтобы быть заметными.
В прошлой статье мы рассматривали это преобразование с точки зрения умножения матрицы на некоторый вектор.
Да, это хороший подход к рассуждениям, но гораздо проще представить кодирование сигнала в виде наглядной схемы.

Попробуем сперва разобраться на примере.
Предположим, что мы работаем с последовательностью чисел.
Итак, мы хотим разбить сигнал на две части вдвое меньшего размера: низкочастотную и высокочастотную.
Тут мы можем воспользоваться особенностью реальных изображений, которая отличает их от шума.
_Значения соседних пикселей обычно близки._

Есть ещё один важный момент.
Если мы планируем потом применять преобразование повторно к низкочастотной части, _нужно сохранять среднее_.
То есть, среднее значение исходной последовательности должно быть равно среднему низкочастотной части.
Если оно будет больше или меньше, то после повторных преобразований значения будут всё время увеличивать или уменьшаться.
С практической точки зрения то может привести к переполнению или ошибкам округления.

Попробуем применить эти идеи на практике.
Мы можем выделить общий план взяв полусуммы соседних пикселей (обозначим их $$x$$ и $$y$$), это не изменит общий план и сохранит среднее.
А детали — это отличия между пикселями.
Для выделения отличий мы можем взять разности.
Обозначим эти величины как $$s$$, $$d$$.

$$\begin{aligned}
s &= \frac{x+y}2,\\
d &= y - x.
\end{aligned}$$

Это преобразование обратимо и мы всегда можем получить исходные значения.

$$\begin{aligned}
x &= s + \frac{d}2,\\
y &= s - \frac{d}2.
\end{aligned}$$

Но можно ли такое преобразование производить эффективно, чтобы не заводить дополнительные массивы?

Оказывается, можно.
Мы можем выполнить его в два этапа заменяя элементы исходного массива без создания нового.

1. $$(x, y) \to (x, y - x) = (x, d)$$,
2. $$(x, d) \to (x + \frac{d}2, d) = (x+\frac{y-x}2, d) = (\frac{x+y}2, d) = (s, d)$$.

Аналогично можно записать обратное преобразование
1. $$(s, d) \to (s - \frac{d}2, d) = (s - \frac{y-x}2, d) = (\frac{x+y}2 - \frac{x-y}2, d) = (x, d)$$.
2. $$(x, d) \to (x, d + x) = (x, x - y + x) = (x, y)$$,

То есть, просто выполняем шаги в обратном порядке.

Мы получили вполне рабочий алгоритм для разделения сигнала по частотам.

Лифтинг
=======

Для наглядности представим это в виде диаграммы.
До сих пор мы рассматривали пару значений $$(x, y)$$, но ничто не мешает применить этот же подход ко всем парам.
Для этого разделим входной поток чисел на те, что стоят на чётных ($$x_i$$) и нечётных ($$y_i$$) местах.

![Схема для рассмотренного алгоритма]({{ site.url }}/assets/2020-05-25-lifting-scheme/lifting-example.svg)

Синие прямоугольники — это преобразования, результаты которых мы прибавляем или вычитаем.
Позже мы поговорим об этом подробнее.

С помощью этой схемы сразу понятно, как построить обратное.
Нужно лишь выполнить действия в обратном порядке.

![Схема для прямого и обратного преобразований]({{ site.url }}/assets/2020-05-25-lifting-scheme/lifting-reverse.svg)

Здесь для краткости введены два блока: S (split) для разделения пар значений и M (merge) для объединения.
$$A$$ — это исходная последовательность чисел, $$\bar{A}$$ — реконструированная.
Если мы выполняли все действия без округлений, то $$A=\bar{A}$$.

Из этой схемы легко получить формулы для обратного преобразования, которые мы вывели раньше.

Так как последовательность $$s_i$$ — это, по сути, уменьшенная копия исходных данных, то можно применить к ней те же процедуры.
Получаем каскад из преобразований.

![Схема каскада из преобразований]({{ site.url }}/assets/2020-05-25-lifting-scheme/lifting-cascade.svg)

Заметим, что хотя у всех величин стоит индекс $$i$$, диапазон для этого индекса зависит от номера этапа.
На каждом этапе длины разделяемых последовательностей уменьшаются вдвое.

Пусть последний этап имеет номер $$n$$.
Так как каждый этап обратим, то исходные числа можно восставновить зная

$$
s^{(n)}_i; d^{(n)}_i, d^{(n-1)}_i, \ldots, d^{(1)}_i.
$$

Рассмотренная нами процедура и называется схемой лифтинга.


Предсказание и обновление
=========================

Схема лифтинга — это общий подход.
То есть, функции $$P$$ и $$U$$ могут быть различными, не обязательно такими, как в примере.
Давайте разберёмся, для чего они вообще нужны.

Ещё раз взглянем на один этап схемы.

![Один этап преобразования]({{ site.url }}/assets/2020-05-25-lifting-scheme/lifting-one-stage.svg)

Названия $$P$$ и $$U$$ выбраны не случайно.

Функция $$P$$ (predict, предсказание) предсказывает значения в одной ветке по значениям в другой.
Если мы потом вычтем предсказания из значений, то останутся только высокочастотные детали.
Чем лучше мы предсказываем, тем ближе детали к нулю.

Функция $$U$$ (update, обновление) нужна для коррекции низкочастотной части.
Если её не скорректировать, то не будет сохраняться среднее значение.
Простой пример — это последовательность

$$0, 1, 0, 1, 0, 1, 0, 1.$$

Среднее значение последовательности равно $$\frac12$$, а среднее значение $$x_i$$ будет нулевым.

Для коррекции нужно как-то передать информацию о $$y_i$$.
Для этого и нужно прибавление $$U$$ к $$x_i$$.
В нашем примере после прибавления мы получаем, что $$s_i = \frac{x_i+y_i}2$$.
А это означает, что среднее будет сохраняться.

В самом деле, рассмотрим последовательность из 6 точек:

$$
x_0, y_0, x_1, y_1, x_2, y_2.
$$

Среднее исходной последовательности равно

$$
\frac{x_0 + y_0 + x_1 + y_1 + x_2 + y_2}6.
$$

Среднее $$s_i$$ будет равно

$$
\frac{\frac{x_0 + y_0}2 + \frac{x_1 + y_1}2 + \frac{x_2 + y_2}2}3.
$$

А это то же самое.

Вейвлет CDF 5/3
===============

Запишем ещё раз формулы для нашего примера.

$$
\begin{aligned}
d_i &= y_i - p_i = y_i - x_i,\\
s_i &= x_i + u_i = x_i + \frac12 d_{i}.
\end{aligned}
$$

Здесь $$p_i$$ и $$u_i$$ — это предсказание и обновление соответственно.

Разберёмся с предсказанием.
У нас есть последовательность

$$\ldots, x_{i-1}, y_{i-1}, x_{i}, y_{i}, x_{i+1}, y_{i+1}, \ldots $$

Мы хотим предсказать $$y_i$$ по соседним значениям в последовательности.
Как видно из формул, мы в качестве предсказания просто берём $$x_i$$.
Это отлично работает, если у нас сигнал постоянный или кусочно-постоянный.
Тогда предсказания будут идеально точными, а высокочастотная часть нулевой.

Но константные сигналы встречаются не так часто.
Поступим хитрее, попробуем предсказать сигнал, в предположении, что он кусочно-линейный.
Да, он может не быть таким, но мы всё равно предскажем его точнее и разность будет меньше.

Для линейного сигнала $$y_i$$ будет равен

$$\frac{x_i + x_{i+1}}2.$$

(Если это неочевидно, попробуйте построить график линейной функции и отметить точки $$x_i$$, $$y_i$$ и $$x_{i+1}$$.)

Если мы меняем предсказание, то нужно изменить и обновление.
В качестве функции обновления возмём прежнюю из примера, но заменим $$d_i$$ на среднее соседних разностей.

В итоге получаем следующие выражения

$$
\begin{aligned}
d_i = y_i - \frac{x_i + x_{i+1}}2,\\
s_i = x_i + \frac12 \frac{d_{i-1} + d_{i}}2.
\end{aligned}
$$

Схема преобразования выглядит так:

![Схема преобразования CDF 5/3]({{ site.url }}/assets/2020-05-25-lifting-scheme/lifting-cdf53.svg)

Давайте убедимся, что обновление работает как надо, то есть сохраняет среднее.
Для этого выразим значения $$s_i$$:

$$
\begin{aligned}
s_i &= x_i + \frac12 \frac{d_{i-1} + d_{i}}2 =\\
    &= x_i + \frac14 \left( y_{i-1} - \frac{x_{i-1} + x_{i}}2 + y_i - \frac{x_i + x_{i+1}}2 \right) =\\
    &= -\frac18 x_{i-1} + \frac28 y_{i-1} + \frac68 x_i + \frac28 y_i -\frac18 x_{i+1}.
\end{aligned}
$$

Возьмём линейный сигнал в общем виде:

$$ a, a + h, a + 2h, a + 3h, a + 4h.$$

Несложно посчитать, что его среднее будет равно

$$ \frac{a + a + h + a + 2h + a + 3h + a + 4h}5 = a + 2h.$$

После преобразования получаем

$$ -\frac18 a + \frac28 (a + h) + \frac68 (a + 2h) + \frac28 (a + 3h) -\frac18 (a + 4h) = a + 2h.$$

То есть, среднее для линейного сигнала сохраняется, как нам и нужно.
При этом, значения $$d_i$$ будут нулевым уже не только для константного сигнала, но и для всех линейных участков.

Преобразование с коэффициентами $$\left(-\frac18, \frac28, \frac68, \frac28, -\frac18\right)$$ для фильтра низких частот и $$\left(-\frac12, 1, -\frac12\right)$$ для высоких известно под названием CDF 5/3.
Это вейвлет-преобразование названо по первым буквам фамилий — Коэн, Добеши и Фово.
А цифры — это длины фильтров.

У этого преобразования есть ещё одно замечательное свойство.
Знаменатели всех дробных коэффициентов — это степени двойки.
А так как компьютеры работают в двоичной системе, то работать с такими дробями можно без потерь на округление.
Это стало одной из причин, почему CDF 5/3 используется для сжатия без потерь в формате JPEG 2000 (не путать с обычным JPEG).

Ну и ещё раз заметим, что это преобразование тоже можно выполнить без вспомогательного массива, на месте.

Правда, есть один нюанс.
При вычислении предсказания для последней точки мы выходим за переделы массива, так как элемента $$z_{i+1}$$ уже нет, когда $$z_{i}$$ последний.
Это решается достаточно просто.
Мы можем для элементов за пределами использовать значения, взятые зеркально с края массива.

Пусть номер последнего элемента массива равен $$M - 1$$ (мы нумеруем с нуля).
В качестве $$z_{M}$$ тогда берём $$z_{M-1}$$, в качестве $$z_{M+1}$$ возьмём $$z_{M-2}$$ и так далее.

Аналогично для левого края и отрицательных индексов.

Другие вейвлеты
===============

К тому же никто не ограничает нас только одним шагом предсказания и одним шагом обновления.
Попробуем записать схему для другого известного преобразования — вейвлета Добеши D4.
Мы рассматривали этот вейвлет в прошлой статье.

![Схема преобразования D4]({{ site.url }}/assets/2020-05-25-lifting-scheme/lifting-d4.svg)

Здесь добавились два умножения, нужные для нормализации значений сигналов.
Эти умножения, очевидно, обратимы, поэтому мы всегда можем записать схему обратного преобразования.


Заключение
==========

Итак, мы познакомились с интересным и мощным средством построения вейвлет-преобразований.
Схемы не просто сделали рассуждения нагляднее.
С помощью этих схем преобразование можно выполнять на месте, без использования дополнительных массивов.
Да и в целом программировать глядя на такую схему намного проще.

В одной из следующих статей мы попробуем запрограммировать эти преобразования, и посмотреть, насколько хорошо они позволяют сжимать изображения.

Если интересуют подробности, то кроме литературы из [прошлой статьи]({{ site.url }}/2019/01/31/wavelets-theory/) можно посмотреть:

- [Arne Jensen _An Animated Introduction to the Discrete Wavelet Transform_](https://www.math.aau.dk/digitalAssets/120/120646_r-2003-24.pdf). — Слайды с очень доходчивым объяснением схемы лифтинга с конкретными примерами.
- Usevitch B. [A tutorial on modern lossy wavelet image compression: Foundations of JPEG 2000.](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.6.7887&rep=rep1&type=pdf) Signal Processing Magazine, IEEE. 18. 22 - 35. DOI: 10.1109/79.952803. — Описание устройства формата сжатия JPEG 2000.
