---



---
---

Данная заметка - перевод [вот этой](https://www.songho.ca/opengl/gl_camera.html) статьи.

---
## Обзорная информация

**OpenGL** не определяет явно ни объект _Камеры_, ни какую-то специфическую Матрицу для трансформации _Камеры_. Вместо этого **OpenGL** трансформирует целую сцену (включая и камеру) обратно к пространству координат, где фиксированая Камера находится в центре координат $(\textcolor{#FF0000}{0}, \textcolor{#01FF00}{0}, \textcolor{#0000FF}{0}, 0)$﻿ и всегда смотрит по оси $\textcolor{#0000FF}{-Z}$﻿. Это пространство и называется _**Пространством Наблюдателя**_.

Как раз-таки поэтому OpenGL используте единую _Матрицу Модель/Представление_ для обеих, и объектной трансформации в _**Мировое Пространство**_, и трансформация Камеры (_Пердставления_) в _**Координаты Наблюдателя**_.

Данную трансформацию можно разбить на две логичные подматрицы:  
  
$$M_{\text{ModelView}} \ =\ M_{\text{View}} M_{\text{Model}}$$
![[gl_camera01.png]]

Камера в OpenGL всегда находится в центре координат и смотрит по оси $\textcolor{#0000FF}{-Z}$﻿ в _**Пространстве Наблюдателя**_.

Это означает, что в начале, каждый объект на сцене трансформируется при помощи своей собственной $M_{\text{Model}}$﻿, а потом целая сцена трансформируется обратно (_reversly_) при помощи $M_{\text{View}}$﻿. В этой заметке му будем обсуждать только $M_{\text{View}}$﻿ для трансформации _Камеры_.
## $LookAt$﻿

Функция [gluLookAt()](http://www.opengl.org/sdk/docs/man2/xhtml/gluLookAt.xml) используется для создания матрицы представления, где камера расположена в позиции наблюдателя $(\textcolor{#FF0000}{x_{\text{eye}}}, \textcolor{#01FF00}{y_{\text{eye}}}, \textcolor{#0000FF}{z_{\text{eye}}}, w_{\text{eye}})$﻿ и смотрит на (looking at), либо вращается, к целевой точке $(\textcolor{#FF0000}{x_{\text{t}}}, \textcolor{#01FF00}{y_{\text{t}}}, \textcolor{#0000FF}{z_{\text{t}}}, w_{\text{t}})$﻿. И Позиция Наблюдателя и Целевая точка определены в _**Мировых Координатах**_. В данной секции описывается как реализовать матрицу, аналогичную [gluLookAt()](http://www.opengl.org/sdk/docs/man2/xhtml/gluLookAt.xml), самостоятельно.

Трансформация _Камеры_ $lookAt$﻿ состоит из двух частных трансформций:

1. Транслирование всей _Сцены_ в обратном направлении с _Позиции Наблюдателя_ к началу координат ($M_{T}$﻿).
2. Вращение сцены с обратной ориентацией ($M_{R}$﻿).

Эти две трансформации делаются так, что бы камера в итоге была размещена в Центре Координат и смотрела по оси $\textcolor{\#0000FF}{-Z}$﻿.

Соответственно объединение трансформаци 1 и 2 будет выглдядить следующим образом:

$$
M_{\text{View}} \ =\ M_{R}M_{T} \ =\ \begin{pmatrix} \textcolor{#FF0000}{r_{0}} && \textcolor{#01FF00}{r_{4}} && \textcolor{#0000FF}{r_{8}} && 0 \\ \textcolor{#FF0000}{r_{1}} && \textcolor{#01FF00}{r_{5}} && \textcolor{#0000FF}{r_{9}} && 0 \\ \textcolor{#FF0000}{r_{2}} && \textcolor{#01FF00}{r_{6}} && \textcolor{#0000FF}{r_{10}} && 0 \\ 0 && 0 && 0 && 1 \end{pmatrix} \begin{pmatrix} \textcolor{#FF0000}{1} && 0 && 0 && \textcolor{#FF0000}{t_{x}} \\ 0 && \textcolor{#01FF00}{1} && 0 && \textcolor{#01FF00}{t_{y}} \\ 0 && 0 && \textcolor{#0000FF}{1} && \textcolor{#0000FF}{t_{z}} \\ 0 && 0 && 0 && 1 \end{pmatrix} 
\ =\
$$$$
\ =\ 
\begin{pmatrix} \textcolor{#FF0000}{r_{0}} && \textcolor{#01FF00}{r_{4}} && \textcolor{#0000FF}{r_{8}} && \textcolor{#FF0000}{r_{0}} \textcolor{#FF0000}{t_{x}} + \textcolor{#01FF00}{r_{4}} \textcolor{#01FF00}{t_{y}} + \textcolor{#0000FF}{r_{8}} \textcolor{#0000FF}{t_{z}} \\ \textcolor{#FF0000}{r_{1}} && \textcolor{#01FF00}{r_{5}} && \textcolor{#0000FF}{r_{9}} && \textcolor{#FF0000}{r_{1}} \textcolor{#FF0000}{t_{x}} + \textcolor{#01FF00}{r_{5}} \textcolor{#01FF00}{t_{y}} + \textcolor{#0000FF}{r_{9}} \textcolor{#0000FF}{t_{z}} \\ \textcolor{#FF0000}{r_{2}} && \textcolor{#01FF00}{r_{6}} && \textcolor{#0000FF}{r_{10}} && \textcolor{#FF0000}{r_{2}} \textcolor{#FF0000}{t_{x}} + \textcolor{#01FF00}{r_{6}} \textcolor{#01FF00}{t_{y}} + \textcolor{#0000FF}{r_{10}} \textcolor{#0000FF}{t_{z}} \\ 0 && 0 && 0 && 1 \end{pmatrix}
$$

Предположим, что Камера находится в $( \textcolor{#FF0000}{2}, \textcolor{#01FF00}{0}, \textcolor{#0000FF}{3} )$﻿ и смотрит в $( \textcolor{#FF0000}{0}, \textcolor{#01FF00}{0}, \textcolor{#0000FF}{0} )$﻿ в Мировых Координатах. Для того что бы построить Матрицу Представления для данного кейса - нам надо транслировать Мир (типа все объекты в _**Мировом Пространстве**_) в координату $( \textcolor{#FF0000}{-2}, \textcolor{#01FF00}{0}, \textcolor{#0000FF}{-3} )$﻿ и повернуть на $-33.7^{\circ}$﻿ вдоль оси $\textcolor{#01FF00}{Y}$﻿. И как результат, камера начинает смотреть вдоль оси $\textcolor{#0000FF}{-Z}$﻿, находясь при этом в начале координат.

![[gl_camera02.gif]]

Пример трансформации камеры в **OpenGL** при помощи `lookAt()`

Первая часть, трансляционная, матрицы $lookAt$﻿ выглядит довольно просто. Мы просто двигаем _Камеру_ от изначальной её позиции в начало координат. Матрица трансляции $M_{T}$﻿, после замены 4ой колонки отрицанием _**Позиции Наблюдателя**_, будет выглядеть так:
$$
M_{T} \ =\ \begin{pmatrix} \textcolor{#FF0000}{1} && 0 && 0 && -\textcolor{#FF0000}{x_{eye}} \\ 0 && \textcolor{#01FF00}{1} && 0 && -\textcolor{#01FF00}{y_{eye}} \\ 0 && 0 && \textcolor{#0000FF}{1} && -\textcolor{#0000FF}{z_{eye}} \\ 0 && 0 && 0 && 1 \end{pmatrix}
$$

Часть матрицы поворота $M_{R}$﻿ матрицы $lookAt$﻿ - гораздо тяжелее, чем трансляция, потому что нам придется вычислять первую, вторую и третью колонку матрицы поворота.

![[gl_camera05.png]]

$\overrightarrow{\textcolor{#FF0000}{левый}}$﻿, $\overrightarrow{\textcolor{#01FF00}{верхний}}$﻿ и $\overrightarrow{\textcolor{#0000FF}{передний}}$﻿ вектора, показывающие направление от цели $v_{t}$﻿ к _**Координатам Наблюдателя.**_
Во-первых высчитаем нормализованый $\overrightarrow{\textcolor{#0000FF}{передний}}$﻿ вектор $\overrightarrow{\textcolor{#0000FF}{f}}$﻿, который идет от целевой (_target_) точки $v_{t}$﻿ в сторону позиции _Наблюдателя_ $v_{e}$﻿.

Причем важно обратить внимание, что вектор $\overrightarrow{\textcolor{#0000FF}{f}}$﻿ идет от $v_{t}$﻿ к $v_{e}$﻿, а не наоброт - это происходит потому что на самом деле сцена перевернута, а не _Камера_.

Соответветсвенно данное расстояние предстваим как:
$$
v_{e} - v_{t} \ =\ ( \textcolor{#FF0000}{x_{e}} - \textcolor{#FF0000}{x_{t}}, \textcolor{#01FF00}{y_{e}} - \textcolor{#01FF00}{y_{t}}, \textcolor{#0000FF}{z_{e}} - \textcolor{#0000FF}{z_{t}} )
$$
А соответсвенно единичный вектор $\overrightarrow{\textcolor{#0000FF}{f}}$﻿ будет равен:
$$
\overrightarrow{\textcolor{#0000FF}{f}} \ =\ \frac{ v_{e} - v_{t} } { ||v_{e} - v_{t}|| }
$$
Во-вторых, высчитаем нормализованый $\overrightarrow{\textcolor{#FF0000}{левый}}$﻿ вектор $\overrightarrow{\textcolor{#FF0000}{l}}$﻿ при помощи векторного произведения с данным $\overrightarrow{\textcolor{#01FF00}{верхним}}$﻿ вектом $\overrightarrow{\textcolor{#01FF00}{u}}$﻿. Если вдруг данный вектор не задан, то надо просто взять либо вектор, соблюдающий ортогональность к плоскости образующейся от векторов $\overrightarrow{\textcolor{#0000FF}{f}}$﻿ и $\overrightarrow{\textcolor{#FF0000}{l}}$﻿, к примеру это может быть $($﻿, если предположить что камера расположена строго параллельно оси $\overrightarrow{\textcolor{#01FF00}{+Y}}$﻿.
Соотвественно, получим вектор:
$$
\overrightarrow{\textcolor{#FF0000}{left}} \ =\ \overrightarrow{\textcolor{#01FF00}{u}} \times \overrightarrow{\textcolor{#0000FF}{f}}
$$
А значит в нормализованном виде, это будет выглядеть как:
$$
\overrightarrow{\textcolor{#FF0000}{l}} \ =\ \frac { \textcolor{#FF0000}{left} } { ||\textcolor{#FF0000}{left}|| }
$$
И наконец-то в-третьих, мы высчитываем номализованный $\overrightarrow{\textcolor{#01FF00}{верхний}}$﻿ вектор, при помощи всех прошлых вычислений. Мы так же берем векторное выражение $\overrightarrow{\textcolor{#FF0000}{левого}}$﻿ и $\overrightarrow{\textcolor{#0000FF}{переднего}}$﻿, а значит все эти три вектора ==ортонормальны== (Это значит, что они и Ортоганальны и Нормальны). Обратим внимание, что мы не высчитываем в ручную нормализацию вектора $\overrightarrow{\textcolor{#01FF00}{u}}$﻿, так как векторное произведение двух единичных векторов как раз и дает единичный вектор:
$$
\overrightarrow{\textcolor{#01FF00}{u}} \ =\ \overrightarrow{\textcolor{#0000FF}{f}} \times \overrightarrow{\textcolor{#FF0000}{l}}
$$
Эти три базовых вектора $\overrightarrow{\textcolor{#01FF00}{u}}, \overrightarrow{\textcolor{#0000FF}{f}}, \overrightarrow{\textcolor{#FF0000}{l}}$﻿ как раз-таки и нужны для конструирования матрицы поворота $M_{R}$﻿. Однако мы не должны забывать, про то, что матрица должна быть инвертирована.  
Предположим, что  
_Камера_ расположена ровно над сценой, тогда целая сцена должна повернуться вниз инвертировано, что бы камера могла смотреть по направлению $\textcolor{#0000FF}{-Z}$﻿.

![[gl_camera03.gif]]

Сцена вращается вниз, если камера Сверху.

Ровно по такому же принципу, если _Камера_ расположена слева на сцене, то вся сцена должна повернуться Направо, для того, что бы камера могла смотреть по направлению $\textcolor{#0000FF}{-Z}$﻿.

![[gl_camera04.gif]]

Сцена вращается направо (по часовой стрелке), если камера идет влево.

Следовательно, если цель вращающейся части $M_{R}$﻿ матрицы $lookAt$﻿ состоит в том, что бы вычислить _Матрицу Вращения_ из _Цели_ в _**Координаты Наблюдателя**_ то следует её инвертировать. И стоит помнить, что инвертированная матрица равняется её транспонированной матрице, из-за ортонормальности (там где вектора ортогональны между собой и каждый вектор - единичный). Выглядит это так:
$$
M_{R} \ =\ \begin{pmatrix} \textcolor{#FF0000}{l_{x}} && \textcolor{#01FF00}{u}_{\textcolor{#FF0000}{x}} && \textcolor{#0000FF}{f}_{\textcolor{#FF0000}{x}} && 0 \\ \textcolor{#FF0000}{l} _{\textcolor{#01FF00}{y}} && \textcolor{#01FF00}{u}_{\textcolor{#01FF00}{y}} && \textcolor{#0000FF}{f}_{\textcolor{#01FF00}{y}} && 0 \\ \textcolor{#FF0000}{l} _{\textcolor{#0000FF}{z}} && \textcolor{#01FF00}{u}_{\textcolor{#0000FF}{z}} && \textcolor{#0000FF}{f}_{\textcolor{#0000FF}{z}} && 0 \\ 0 && 0 && 0 && 1 \end{pmatrix}^{-1} \ =\ \begin{pmatrix} \textcolor{#FF0000}{l_{x}} && \textcolor{#01FF00}{u}_{\textcolor{#FF0000}{x}} && \textcolor{#0000FF}{f}_{\textcolor{#FF0000}{x}} && 0 \\ \textcolor{#FF0000}{l} _{\textcolor{#01FF00}{y}} && \textcolor{#01FF00}{u}_{\textcolor{#01FF00}{y}} && \textcolor{#0000FF}{f}_{\textcolor{#01FF00}{y}} && 0 \\ \textcolor{#FF0000}{l} _{\textcolor{#0000FF}{z}} && \textcolor{#01FF00}{u}_{\textcolor{#0000FF}{z}} && \textcolor{#0000FF}{f}_{\textcolor{#0000FF}{z}} && 0 \\ 0 && 0 && 0 && 1 \end{pmatrix}^{T} \ =\ \begin{pmatrix} \textcolor{#FF0000}{l_{x}} && \textcolor{#FF0000}{l} _{\textcolor{#01FF00}{y}} && \textcolor{#FF0000}{l} _{\textcolor{#0000FF}{z}} && 0 \\ \textcolor{#01FF00}{u}_{\textcolor{#FF0000}{x}} && \textcolor{#01FF00}{u}_{\textcolor{#01FF00}{y}} && \textcolor{#01FF00}{u}_{\textcolor{#0000FF}{z}} && 0 \\ \textcolor{#0000FF}{f}_{\textcolor{#FF0000}{x}} && \textcolor{#0000FF}{f}_{\textcolor{#01FF00}{y}} && \textcolor{#0000FF}{f}_{\textcolor{#0000FF}{z}} && 0 \\ 0 && 0 && 0 && 1 \end{pmatrix}
$$
И соответственно, в конечном итоге, матрица вида для трансформации координат из Объектного Пространства в Пространство Наблюдателя, будет выглядеть так:
$$
M_{\text{View}} = M_{R}M_{T} \ =\ \begin{pmatrix} \textcolor{#FF0000}{l_{x}} && \textcolor{#FF0000}{l} _{\textcolor{#01FF00}{y}} && \textcolor{#FF0000}{l} _{\textcolor{#0000FF}{z}} && 0 \\ \textcolor{#01FF00}{u}_{\textcolor{#FF0000}{x}} && \textcolor{#01FF00}{u}_{\textcolor{#01FF00}{y}} && \textcolor{#01FF00}{u}_{\textcolor{#0000FF}{z}} && 0 \\ \textcolor{#0000FF}{f}_{\textcolor{#FF0000}{x}} && \textcolor{#0000FF}{f}_{\textcolor{#01FF00}{y}} && \textcolor{#0000FF}{f}_{\textcolor{#0000FF}{z}} && 0 \\ 0 && 0 && 0 && 1 \end{pmatrix} \begin{pmatrix} \textcolor{#FF0000}{1} && 0 && 0 && -\textcolor{#FF0000}{x_{eye}} \\ 0 && \textcolor{#01FF00}{1} && 0 && -\textcolor{#01FF00}{y_{eye}} \\ 0 && 0 && \textcolor{#0000FF}{1} && -\textcolor{#0000FF}{z_{eye}} \\ 0 && 0 && 0 && 1 \end{pmatrix} =
$$$$\ =\ \begin{pmatrix} \textcolor{#FF0000}{l_{x}} && \textcolor{#FF0000}{l} _{\textcolor{#01FF00}{y}} && \textcolor{#FF0000}{l} _{\textcolor{#0000FF}{z}} && - \textcolor{#FF0000}{l_{x}} \textcolor{#FF0000}{x_{eye}} - \textcolor{#FF0000}{l} _{\textcolor{#01FF00}{y}} \textcolor{#01FF00}{y_{eye}} - \textcolor{#FF0000}{l} _{\textcolor{#0000FF}{z}} \textcolor{#0000FF}{z_{eye}} \\ \textcolor{#01FF00}{u}_{\textcolor{#FF0000}{x}} && \textcolor{#01FF00}{u}_{\textcolor{#01FF00}{y}} && \textcolor{#01FF00}{u}_{\textcolor{#0000FF}{z}} && - \textcolor{#01FF00}{u}_{\textcolor{#FF0000}{x}} \textcolor{#FF0000}{x_{eye}} - \textcolor{#01FF00}{u}_{\textcolor{#01FF00}{y}} \textcolor{#01FF00}{y_{eye}} - \textcolor{#01FF00}{u}_{\textcolor{#0000FF}{z}} \textcolor{#0000FF}{z_{eye}} \\ \textcolor{#0000FF}{f}_{\textcolor{#FF0000}{x}} && \textcolor{#0000FF}{f}_{\textcolor{#01FF00}{y}} && \textcolor{#0000FF}{f}_{\textcolor{#0000FF}{z}} && \textcolor{#0000FF}{f}_{\textcolor{#FF0000}{x}} \textcolor{#FF0000}{x_{eye}} - \textcolor{#0000FF}{f}_{\textcolor{#01FF00}{y}} \textcolor{#01FF00}{y_{eye}} - \textcolor{#0000FF}{f}_{\textcolor{#0000FF}{z}} \textcolor{#0000FF}{z_{eye}} \\ 0 && 0 && 0 && 1 \end{pmatrix}
$$
## Поворот Камеры (Тангаж(Pitch)-Рыскание(Yaw)-Крен(Roll))

Еще один тип движения камеры - это её поворот. В том числе сейчас мы рассмотрим поворот в контексте углов Эйлера, это будет следующая тройка Тангаж-Рыскание-Крен.

- _**Тангаж**_ - вращение камеры вверх и вниз, вокруг оси $\textcolor{#FF0000}{+X}$﻿ (вращение вокруг $\textcolor{#FF0000}{левой}$﻿ оси).
- _**Рыскание**_ - вращение влево и вправо, вокруг оси $\textcolor{#01FF00}{+Y}$﻿ (вращение вокруг $\textcolor{#01FF00}{верхней}$﻿ оси).
- _**Крен**_ - вращение камеры относительно того, куда кумера смотрит, вокрун оси $\textcolor{#0000FF}{+Z}$﻿ (вращение вокруг $\textcolor{#0000FF}{прямой}$﻿ оси).

Такие углы как раз испольщуются для камеры свободного полета, при движении в VR шлеме, либо же камера игрока от первого лица в шутерах.

Первым делом, нам надо будет свинуть целую Сцену в обратном направлении, относительно позиции Камеры в центре координат $(\textcolor{#FF0000}{0}, \textcolor{#01FF00}{0}, \textcolor{#0000FF}{0})$﻿, так же как мы это ранее делали в функции [[OpenGL Камера. Математическое Представление]]. Потом надо повернуть сцену относительно осей вращения $\textcolor{#FF0000}{+X}$﻿, $\textcolor{#01FF00}{+Y}$﻿, $\textcolor{#0000FF}{+Z}$﻿ независимо.

Если за раз сразу несколько поворотов применяются, то комбинировать их следует последовательно. Так как разная последовательность трансформаций вращения - приводит к разному результату. Самая частая последовательность поворота это _**Крен**_ ($\textcolor{#0000FF}{+Z}$﻿) $\rightarrow$﻿ _**Рыскание**_ ($\textcolor{#01FF00}{+Y}$﻿) $\rightarrow$﻿ _**Тангаж**_ ($\textcolor{#FF0000}{+X}$﻿).

Не забываем так же, что о направлениях вращения (Правило правой руки), положительный угол _**Тангажа**_ вращает камеру вниз, положительный угол Рыскания - вращает камеру влево и положительный угол _**Крена**_ вращает камеру по часовой стрелке (_clockwise_).

### Поворот камеры применяя Кватернионы

Если вдруг для представления вращения используются [[Из Кватернионов к Матрице Вращения]], то углы _**Крен**_, _**Рыскание**_ и _**Тангаж**_ могут быть конвертированы в кватернионное представление из Углов Эйлера (вращения вокруг осей $\textcolor{#FF0000}{+X}$﻿, $\textcolor{#01FF00}{+Y}$﻿, $\textcolor{#0000FF}{+Z}$﻿) и угла, насколько повернулось значение:

![[basic_quaternion.svg]]

Поворот через кватернионы вокруг оси на заданный угол
$$
\begin{split} q &= cos\ \theta + \overrightarrow{r}\ sin\ \theta \\ &= cos\ \theta\ + r_{\textcolor{#FF0000}{x}}sin\ \theta\ + r_{\textcolor{#01FF00}{y}}sin\ \theta\ + r_{\textcolor{#0000FF}{z}}sin\ \theta\ \end{split}
$$
А потом просто будем умножать кватернионы один за другим, что бы создать последовательность вращения. Более подробно все преобразования Кватернионов собраны в [[Из Кватернионов к Матрице Вращения]].

## Камера: Сдвиги и Движение

Для того, что бы обновить позицию, где находится _Наблюдатель_ (_Камера_) при помощи сдвига влево/вправо, либо при движении Вверх/Вниз или Вперед/Назад - нам потребуется найти 3 локальных осевых вектора: $\overrightarrow{\textcolor{#FF0000}{левый}}$﻿, $\overrightarrow{\textcolor{#01FF00}{верхний}}$﻿ и $\overrightarrow{\textcolor{#0000FF}{передний}}$﻿ - они будут расположены в точке, что считается центром для Камеры. При этом они не могут быть ровно такими же как и базовые оси $\textcolor{#FF0000}{+X}$﻿, $\textcolor{#01FF00}{+Y}$﻿, $\textcolor{#0000FF}{+Z}$﻿ _**Мировых Координат**_, из-за того, что виртуальная (мнимая) Камера будет ориентирована в произвольном направлении.

![[gl_camera09.jpg]]

Локальные вектора $\overrightarrow{\textcolor{#FF0000}{левый}}$﻿, $\overrightarrow{\textcolor{#01FF00}{верхний}}$﻿ и $\overrightarrow{\textcolor{#0000FF}{передний}}$﻿ для _Камеры_.

Если нам известна целевая точка, кула смотриь Камера, а также позиция Камеры, мы можем высчитать локальные координаты при помощи разницы и векторного произведения (По аналогии со всеми функциями $lookAt$﻿).

Либо же эти координаты можно достать напрямую из Матрицы вида. Первый ряд отвечает за $\overrightarrow{\textcolor{#FF0000}{левый}}$﻿ вектор. Второй ряд - это $\overrightarrow{\textcolor{#01FF00}{верхний}}$﻿ и третий ряд - $\overrightarrow{\textcolor{#0000FF}{передний}}$﻿.
$$
M_{\text{View}} \ =\ \begin{pmatrix} \textcolor{#FF0000}{l_{x}} && \textcolor{#FF0000}{l} _{\textcolor{#01FF00}{y}} && \textcolor{#FF0000}{l} _{\textcolor{#0000FF}{z}} && - \textcolor{#FF0000}{l_{x}} \textcolor{#FF0000}{x_{eye}} - \textcolor{#FF0000}{l} _{\textcolor{#01FF00}{y}} \textcolor{#01FF00}{y_{eye}} - \textcolor{#FF0000}{l} _{\textcolor{#0000FF}{z}} \textcolor{#0000FF}{z_{eye}} \\ \textcolor{#01FF00}{u}_{\textcolor{#FF0000}{x}} && \textcolor{#01FF00}{u}_{\textcolor{#01FF00}{y}} && \textcolor{#01FF00}{u}_{\textcolor{#0000FF}{z}} && - \textcolor{#01FF00}{u}_{\textcolor{#FF0000}{x}} \textcolor{#FF0000}{x_{eye}} - \textcolor{#01FF00}{u}_{\textcolor{#01FF00}{y}} \textcolor{#01FF00}{y_{eye}} - \textcolor{#01FF00}{u}_{\textcolor{#0000FF}{z}} \textcolor{#0000FF}{z_{eye}} \\ \textcolor{#0000FF}{f}_{\textcolor{#FF0000}{x}} && \textcolor{#0000FF}{f}_{\textcolor{#01FF00}{y}} && \textcolor{#0000FF}{f}_{\textcolor{#0000FF}{z}} && \textcolor{#0000FF}{f}_{\textcolor{#FF0000}{x}} \textcolor{#FF0000}{x_{eye}} - \textcolor{#0000FF}{f}_{\textcolor{#01FF00}{y}} \textcolor{#01FF00}{y_{eye}} - \textcolor{#0000FF}{f}_{\textcolor{#0000FF}{z}} \textcolor{#0000FF}{z_{eye}} \\ 0 && 0 && 0 && 1 \end{pmatrix}
$$
По факту нам нужна только $M_{R}$﻿ часть от $M_{\text{View}}$﻿. Так как часть отвечающая за вращение матрицы $M_{\text{View}}$﻿ обратно-трансформированы (транспонированы), эти вектора становятся векторами ряда, а не колонки. И из-за того, что основные $\textcolor{#FF0000}{+X}$﻿, и $\textcolor{#0000FF}{+Z}$﻿ оси виртуальной Камеры смотрят в противоположенные стороны, мы должны не забыть сделать отрицательными компоненты $\overrightarrow{\textcolor{#FF0000}{левого}}$﻿ и $\overrightarrow{\textcolor{#0000FF}{переднего}}$﻿ вектора (_ось_ _$\overrightarrow{\textcolor{#01FF00}{+Y}}$_﻿ _смотрит в одном направлении и для Мировых Координат и Координат Наблюдателя, поэтому_ _$\overrightarrow{\textcolor{#01FF00}{верхний}}$_﻿ _вектор не нужно модифицировать_).
$$
\begin{split} \begin{cases} \text{Камера } \overrightarrow{\textcolor{#FF0000}{Лево}} \ &=\ ( \textcolor{#FF0000}{l_{x}} ,\ \textcolor{#FF0000}{l} _{\textcolor{#01FF00}{y}} ,\ \textcolor{#FF0000}{l} _{\textcolor{#0000FF}{z}} ) \\ \text{Камера } \overrightarrow{\textcolor{#01FF00}{Верх}} \ &=\ ( \textcolor{#01FF00}{u}_{\textcolor{#FF0000}{x}} ,\ \textcolor{#01FF00}{u}_{\textcolor{#01FF00}{y}} ,\ \textcolor{#01FF00}{u}_{\textcolor{#0000FF}{z}} ) \\ \text{Камера } \overrightarrow{\textcolor{#0000FF}{Вперед}} \ &=\ ( \textcolor{#0000FF}{f}_{\textcolor{#FF0000}{x}} ,\ \textcolor{#0000FF}{f}_{\textcolor{#01FF00}{y}} ,\ \textcolor{#0000FF}{f}_{\textcolor{#0000FF}{z}} ) \end{cases} \end{split}
$$