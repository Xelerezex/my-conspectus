OpenGL - это сценарный, кроссплатформенный, язык программирования являющийся API для отрисовки 2D и 3D графики на экране, через **GPU** (**Graphics Processing Unit**).

OpenGL 2.0 - стала deprecated в районе 2008 года, когда выкатили новую версию OpenGL 3.0, которая полностью меняла пайплайн работы с OpenGL. В версии 2.0 испольовались фикс-функции([fixed-function](https://translated.turbopages.org/proxy_u/en-ru.ru.649f9195-65ad1389-91cc340b-74722d776562/https/en.wikipedia.org/wiki/Fixed-function)), которые надо было дергать и потом что-то отображалось на экране. OpenGL 3.0 же использует другой подход - вертексные буфферы для отсылки сырых данных на GPU партиями, а не по одному как это было с фикс-функциями. Это дало серьезный прирост в перформансе.

Сейчас пайплайн работы с OpenGL 3.0 и старше следующий - пакуем все данные, нужные для отрисовки, в вертексные буферные объекты и отсылаем их большим единым паком на GPU, и через шейдерное программирование даем инструкции **GPU** как обраотать и рассчитать эти данные прямо на **GPU**.

Шейдеры пишутся на C-подобном языке **OpenGL Shading Language** (**GLSL**).

---

Тк новый OpenGL работает как стейт-машина, удобно запомнить что есть два типа функций:

1. _state-setting_ - функции которые задают какие-то первичные параметры, перед переходом в новый стейт.
2. _state-using_ - функции перехода в новый определенный стейт.

Пример:  
  
`glClearColor()` - _state-setting_ функция, задает цвет, который будет применен при очистки буфера (сам буфер функция не чистит).

`glClear()` - _state-using_ функция, очищает буфер и может залить все цветомЮ который ранее был задан в `glClearColor()`.

---

Шейдерная программа обычно включает в себя 3 части:

1. Шейдер Геометрии (опционально). Вычисляет создание геометрии перед передачей данных в вертексный шейдер.
2. Вертексный Шейдер. Данный шейдер обрабатывает позицию и движения вертексов (вершина) перед передачей данных в фрагментный шейдер.
3. Фрагментный Шейдер. Вычисляет и отображает результирующие пиксели на экране.

---

Пример самого простого пайплайна при работе с OpenGL:

1. Создать шейдер и подгрузить его в исполняемую программу.
2. Потом мы дергаем **Vertex Buffer Object** (**VBO**) для хранения позиций вершин, для дальнейшей отправки сразу огромной пачки на **GPU**. В **VBO** также можно хранить значения нормалей, текстурных координат и цвета вершин. В **GPU** можно отослать все, что угодно, при условии, что оно бьется с инпутом внутри шейдерного кода.
3. Потом мы записываем **VBO** в **Vertex Array** **Object** (**VAO**) и посылаем целый **VAO** в **GPU** для обработки и расчетов. В **VAO** можно загнать много разных **VBO**, тк, по сути **VAO** можно представить как обычный список из C++ (представить ментально).

---

## Обзор на пайплайн OpenGL

[[Обзор пайплайна OpenGL]]

## Основные темы

[[Основные темы]]

## Освещение

[[Освещение]]

## Продвинутое изучение OpenGl

[[Продвинутое изучение OpenGl]]