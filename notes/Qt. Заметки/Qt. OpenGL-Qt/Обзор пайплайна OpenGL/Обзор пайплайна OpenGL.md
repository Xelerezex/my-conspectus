---

Данная заметка перевод - [вот этой](http://www.songho.ca/opengl/gl_pipeline.html) статьи.

---

Пайплайн **OpenGL** - предсталяет из себя последовательную серию действий.

Два основных представления данных в **OpenGL** - это Данные Основаннве на Координатх Вершин (Вертексные данные) и Данные Основанные на Информации о Пикселях (Пиксельная информация). Эти ключевые представление, в рамках работы с **OpenGL**, проходят через весь пайплайн преобразований и трансформаций, что бы в конечном счете сформировать Фрейм Буффер (_Frame Buffer_).

Так же OpenGL умеет возвращать данные обратно в приложение, где крутится его основной контекст, это можно видить по пунктирным линиям, ниже на графике.

![[opengl_rendering_pipeline_ru.svg]]

- Картинка в оригинале:
    
    ![[opengl_rendering_pipeline_eng.svg]]
    
      
    

## Списки Отображения

Списки Отображения - это группа комманд в OpenGL, которая может храниться (компилироваться) для последующего выполнения. Все данные вертексов и пикселей могут хранится в Списках Отображения. Данная возможность - позволяет оптимизироватьвыполнение программы, тк все данные закешированы. К примеру, когда программа OpenGL будет работать по сети - можно сократить количество передаваемых данных при помощи этой группы команд. Так как Списки Отображения - часть состояния сервера и находятся на серверной машине, клиентской машине следует отсылать команды и данные лишь раз в серверный Список.

## Вертексные Операции

Каждая вершина и каждая _**Нормальная Координата**_ трансформируется посредствам [[Трансформации в OpenGL]]_**[[Трансформации в OpenGL]]**_ (происходит переход из _**Объектных Координат**_ в _**Координаты Наблюдателя**_). Так же, если применяется рендеринг света - то вычисления света для каждой вершины применяется как раз на этом шаге.

## Сборка Примитивов

После применения всех вертексных трансформаций, все примитивы (точка, линия или полигон) [[Математическа стоящая за Ортографической и Перспективной Проекцией]]_**[[Математическа стоящая за Ортографической и Перспективной Проекцией]]**_. Так же на этом шаге происходит отбрасывание точек, которые не попали в видимую часть _Усеченной Пирамиды Камеры_.

Так же потом происходит [[Гомогенные Координаты]] для дальнейшей отрисовки всей 3D сцены на мониторе (из гомогенных координат на монитор вершины отображаются при помощи _[[Математическа стоящая за Ортографической и Перспективной Проекцией]]_).

И самой последней итерацией будет применить Тестирование на брак (_culling test_), если он включен.

## Операция Переноса Пикселей

После того, как пиксели достаются (или распаковываются) из клиентской памяти, эти данные готовы к масштабированию, смещению, отображению и сжиманию. Все эти операции как раз и называются Операциями Переноса Пикселей. Перенесенные данные, в дальнейшем, либо просто хранятся в памяти для дальнейшего использования, либо растеризируются напрямую в фрагменты.

## Текстурная память

Текстурные изображения загружаются в текстурную память, что бы в дальнейшем намазаться на геометрические объекты, полученные после Сборки Примитивов.

## Растеризация

Это процесс объединения **геометрических данных**, полученных после шага _Сборки Примитивов_, и **пиксельных данных** (текстурных), полученных после шага _Операций Переноса Пикселей_, и преобразование этих данных в единый **Фрагмент**.

Фрагмент - есть двумерный массив, хранящий в себе информацию о цвете, глубине, ширине линий, размере точек и вычислений сглаживания (`GL_POINT_SMOOTH`, `GL_LINE_SMOOTH`, `GL_POLYGON_SMOOTH`). Если глобальны режим затенения в OpenGL выставлен на `GL_FILL`, то тогда внутренние пиксели Примитивов будут закрашиваться именно на этом Шаге.

Каждый фрагмент соответсвует одному пиксели во Фрейм Буффере.

## Фрагментарные Операции

Это последний шаг пайплайна OpenGL, который подразумевает в себе перегон Фрашментов в единый Фрейм Буффер.

Первым делом на этом шаге выполянется Тексельная Генерация: текстурный элемент генерируется их Текстурной Памяти и применяется для каждого Фрагмента. Потом происходит вычисление Тумана (fog calculation). И потом прогоняются несколько фрагментарных тестов в следующей последовательности:

1. Тест Вырезки (Scissor Test).
2. Альфа Тестирование (Alpha Test).
3. Трафаретное Тестирование (Stencil Test).
4. Тестирование Глубины (Depth Test).

Наконец, выполняются блендинг, сглаживание, логическая операция и маскировка с помощью битовой маски, и фактические пиксельные данные сохраняются в Фрейм Буффере.

## Дополнительно

OpenGL может возвращать большинство текущих состояний и выходную информацию с помощью команд `glGet*()` и `glIsEnabled()`. Более того, мы можем считывать прямоугольную область пиксельных данных из Фрейм Буффера, используя `glReadPixels()`, и получать полностью преобразованные вершинные данные, используя `glRenderMode(GL_FEEDBACK)`. Функция `glCopyPixels()` не возвращает пиксельные данные в указанную системную память, а копирует их обратно в другой буфер кадров, например, из переднего буфера в задний буфер.