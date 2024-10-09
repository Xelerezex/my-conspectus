---

### Ресурсы:

1. [Альтенативные аллокаторы памяти](https://habr.com/ru/articles/274827/).
2. [Аллокаторы памяти](https://habr.com/ru/articles/505632/).
3. [Memory Allocators](https://github.com/mtrebi/memory-allocators).

---

## Линейный аллокатор.

Данный аллокатор предполагает отсутствие мелочного освобождения  
выделенных ресурсов памяти. Выделяет новую память линейно,  
один за другим из фиксированного пула.  

![[linear-allocator.svg]]

Так как постоянность локальных данных не важна, и очень часто  
количество данных (они могут быть разного размера),  
приходящих в буфер является ограниченным.  

```C++
\#include <iostream>

struct LinearBuffer
{
    uint8_t *mem;       // Указатель на память в буфере
    uint32_t totalSize; // Полный размер буфера в байтах
    uint32_t offset;    // Сдвиг после очередной аллокации
};

void* linearBufferAlloc(LinearBuffer* buff, uint32_t size)
{
    if (!buff || !size)
    {
        return nullptr;
    }

    uint32_t newOffset = buff->offset + size;

    if (newOffset <= buff->totalSize)
    {
        void* ptr = buff->mem + buff->offset;
        buff->offset = newOffset;

        return ptr;
    }

    return nullptr;
}

int main()
{
    std::cout << std::endl << "EXIT_SUCCESS" << std::endl;
    return EXIT_SUCCESS;
}
```

Применяя битовые операции к значению `offset` во время  
выделения памяти и так можно поддержать выровненное аллоцирование.  

Данный метод аллоцирования памяти может быть очень полезным,  
но стоит помнить, что от размера аллоцируемых данных, которые  
будут размещаться в буфере (а иногда и от порядка размещения),  
можно столкнуться с неиспользуемым пространством между выделенной  
памятью в буфере.  
  
Данный метод подходит для выравниваний небольшого размера. Но  
когда идет выделение от одного Мб может случить так, что  
порядок размещения в линейном аллокаторе может оказать радикальное  
влияние на количество неиспользуемой памяти.  
  
Для сброса аллокатора стоит выставить значение  
`offset` в ноль.  
  
Стоит помнить, что у клиентов аллокатора ни в коем случае  
не должно быть указателей на деаллоцируемую память, иначе можно  
сломать аллокатор. (В С++ придется в ручную вызывать деструкторы).  

---

## Стековый аллокатор.

(Не относится на прямую к стеку вызовов)

Линейный аллокатор часто является достаточным, для закрытия многих  
задач выделения памяти. Но если хочется больше контроля над  
тем как ресурсы освобождаются - для этого прекрасно подойдет  
Стековый Аллокатор.  
  
В примере с Линейным Аллокатором упоминалось, что для его очистки,  
нужно выставить значение  
`offset` в ноль.  
И вот именно установка  
**конкретного** значения `offset` в направлении  
обратным направлению прироста заключается ключевая идея Стекового  
Аллокатора.  
  
С каждым куском памяти, выделенным нашим стековым аллокатором  
будет опционально проассоциирован указатель на состояние  
стекового аллокатора непосредственно перед аллоцированием.  
Это значит, что если мы восстановим это состояние аллокатора,  
изменяя его  
`offset`, мы фактически освободим его память, которую  
потом можно будет переиспользовать.  

  
  

![[stack-allocator.svg]]

  

Потом мы производим очистку второго элемента стека:

  

![[stack-allocator-2.svg]]

  

Данная техника создания Аллокаторов полезна при выделении памяти  
под данные с ограниченным временем жизни. К примеру ограниченное  
время жизни функции или подсистемы. Такая стратегия так же  
полезна для таких вещей как ресурсы уровня, у которых четко  
определен порядок освобождения (обратный порядку размещения).  

```C++
\#include <iostream>

struct StackBuffer
{
    uint8_t *mem;       // Указатель на память в буфере
    uint32_t totalSize; // Полный размер буфера в байтах
    uint32_t offset;    // Сдвиг после очередной аллокации
};

using StackHandle = uint32_t;

void* StackBufferAlloc(StackBuffer* buff, uint32_t size, StackHandle* Handle)
{
    if (!buff || !size)
    {
        return nullptr;
    }

    const uint32_t currOffset = buff->offset;
    if (currOffset + size <= buff->totalSize)
    {
        uint8_t *pointer = buff->mem + currOffset;
        return (void*)pointer;
    }

    return nullptr;
}

void stackBufferSet(StackBuffer* buff, StackHandle handle)
{
    buff->offset = handle;
}

int main()
{
    std::cout << std::endl << "EXIT_SUCCESS" << std::endl;
    return EXIT_SUCCESS;
}
```

---

## Двусторонний стековый аллокатор.

Принцип такой же, как и стекового аллокатора, за исключением того,  
что в данном аллокаторе два стека. Один растет снизу вверх.  
Другой растет сверху вниз.  

![[double-sided-stack-allocator.svg]]

Такой аллокатор хорошо использовать в ситуациях, когда  
есть данные схожего типа, но с совершенно разными временами жизни.  
Или если есть данные, которые должны быть тесно связаны и должны  
быть размещены в одной и той же области памяти, но они имеют  
разный размер.  

Стоит помнить, что место, где встретятся смещения двух стеков, не  
фиксировано. Один стек может вырасти в разы больше второго и  
это нормально.  

---

## Пуловый аллокатор.

Принцип работы прошлых трех аллокаторов был основан на линейных  
перемещениях указателей или смещениях. Теперь перейдем к немного  
другому принципу работы.  

Пуловый аллокатор предназначен, в первую очередь, для работы с данными одинакового типа и размера.

Данный аллокатор делит подконтрольный ему буфер памяти на сегменты одинакового размера, а затем позволяет клиенту по его желанию выделять эти куски.

![[pool-allocator-1.svg]]

А для предоставлению пользователю свободных сегментов постоянно надо отслеживать их наличие в пуле.

Есть вариант реализации через стек индексов (или указателей) на свободные сегменты, но для этого часто нужно дополнительное свободное место.

Но сейчас мы опишем другой подход к реализации, через связный список.

Выглядеть это будет следующим образом: Есть служебная структура аллокатора, она хранится в самом начале, имеет всего два поля, одно из которых, это указатель на связный список, и второе поле - указатель на память в буфере.

Сам же связный список размазан по всему пулу, занимая тоже пространство, что и свободные сегменты в буфере памяти.

При инициализации аллокатора мы проходим через все сегменты в буфере, и в начале каждого сегмента записываем указатель с адресом на следующий свободный сегмент. При этом заголовок будет указывать на первый элемент связного списка.

![[pool-allocator-2.svg]]

Для того, что бы выделить память из пула, нужно проделать следующие действия:

1. Передвинуть указатель на голову связного списка в Заголовочной структуре на следующий элемент.
2. Вернуть указатель на элемент, где изначально стояла Заголовочная структура.

Получается, что при выделении памяти мы всегда возвращаем первый элемент в связном списке.

Аналогично, при освобождении сегмента и возвращении его в пул, мы просто вставляем его в голову списка.

Вставка освобождаемых элементов в Голову, а не в Хвост имеет несколько преимуществ:

- Не нужно проходить каждый раз, целый связный список (Или не нужно хранить дополнительный указатель на конец списка).
- У узла, который только что освободили, больше шансов остаться в кеше при последующих размещениях.

Пул, после нескольких выделений и освобождений памяти:

![[pool-allocator-3.svg]]

```C++
\#include <iostream>
\#include <cstdint>

struct Pool
{
    uint8_t *mem;       // Указатель на память в буфере
		uint8_t *head;      // Указатель на связный список
};

// Аллокация одного куска из пула
void* poolAlloc(Pool* buff)
{
		if (!buff)
		{
				return nullptr;
		}

		if (!buff->head)
		{
				return nullptr; // Закончилась память в буфере
		}

		uint8_t* currPtr = buff->head;
		buff->head = (*((uint8_t**)(buff->head)));
		return currPtr;
}

// Возврат куска обратно в пул
void poolFree(Pool* buff, void* ptr)
{
		if (!buff || !ptr)
		{
				return;
		}

		*((uint8_t**)ptr) = buff->head;
		buff->head = (uint8_t*)ptr;
		return;
}

// Инициализация Заголовочной структуры пула, и очистка всех кусков пула
void poolInit(Pool* buff, uint8_t* mem, uint32_t size, uint32_t chunkSize)
{
		if (!buff || !mem || !size || !chunkSize)
		{
				return;
		}

		const uint32_t chunkCount = (size / chunkSize) - 1;
		for (uint32_t chunkIndex = 0; chunkIndex < chunkCount; ++chunkIndex)
		{
				uint8_t* currChunk = mem + (chunkIndex * chunkSize);
				*((uint8_t**)currChunk) = currChunk + chunkSize;
		}

		*((uint8_t**)&mem[chunkCount * chunkSize]) = NULL; // завершающий NULL
		buff->mem = buff->head = mem;
		return;
}

int main()
{
    std::cout << std::endl << "EXIT_SUCCESS" << std::endl;
    return EXIT_SUCCESS;
}
```

---

---

# Аллокаторы и C++.

---

## Основная информация:

Концепутально можно выделить пять основных операций, которых можно осуществить над аллокаторами:

- `create` - создает аллокатор и отдает ему в распоряжение некоторый объем памяти.
- `allocate` - выделяет блок определенного размера из области памяти, которым распоряжается аллокатор.
- `deallocate` - освобождает определенный блок.
- `free` - освобождает все выделенные блоки из памяти аллокатора. Память, выделенная аллокатору, не освобождается.
- `destroy` - уничтожает аллокатор с последующим освобождением памяти, выделенной аллокатору.

---

## Линейный аллокатор:

Идея Линейного Аллокатора состоит в том чтобы:

- Сохранить указатель на начало блока памяти выделенному аллокатору.
- Перемещать второй указатель (или некое числовое представление), когда аллокация будет завершена.

Сразу покажем данную идею на практике.

Суммарно, мы имеем три указателя, которые нужны для реализации Линейного Аллокатора:

1. `start` - начало памяти аллокатора.
2. `used` - конец аллоцированной памяти внутри аллокатора (так же это может быть не указатель, а числовое значение, которое будет представлять собой сдвиг по памяти от начала до конца аллоцированной памяти)
3. `end` - конец аллокатора или другими словами, общий объем памяти (так же это может быть не указатель, а числовое значение, которое будет представлять собой сдвиг по памяти от начала до конца аллокатора)

Представим блок памяти в 14 байт, с тремя, описанными выше, указателями.

![[linear-allocator-1.svg]]

Представим, что в аллокатор поступил запрос на выделение 4 байт памяти.

Действия аллокатора будут следующими:

- Аллокатор проверяет достаточно ли памяти для выделения.
- Сохранить текущее значение указателя `used`, что бы потом отдать его клиенту, который будет использовать этот блок (Это будет указатель на начало блока, который только что выделили).
- Сместить указатель `used` на величину объема выделенного блока памяти. (В нашем примере это 4 байта).

![[linear-allocator-2.svg]]

Далее мы отправляем запрос на аллокацию 8 байт. И соответственно действия аллокатора будут ровно такими же.

![[linaer-allocator-3.svg]]

У нас осталось только два байта в аллокаторе, и состоит рассмотреть два способа, как эту память проаллоцировать:

1. Первый способ заключается в том, что бы просто проаллоцировать 1 байт без всяких выравниваний.
    
    ![[linear-allocator-4.svg]]
    
2. Второй способ аллокации подразумевает в себе выравнивание. Так как мы пытаемся выделить отрезок не кратный 2, имеет смысл выделять +1 байт памяти, что бы не было фрагментации.
    
    ![[linear-allocator-5.svg]]
    

### Освобождение памяти в Линейном Аллокаторе:

Как упоминалось ранее, данный тип аллокаторов не поддерживает выборочное удаление блоков памяти. Если провести аналогию с `malloc()/free()` , то имея указатель на **0xFFAA00**, мы могли бы удалить первый выделенный блок памяти. Но вот линейный аллокатор не может нам этого позволить. И если и очищать что-то в нем, то только полностью:

![[linear-allocator-1.svg]]

---

## Пулловый аллокатор:

Или его также называют блочным аллокатором. Основная идея заключается в том что бы единоразово взять большой кусок памяти и разбить его на большое количество **мелких, равных по размеру** участков. После разделения, данный аллокатор по запросу возвращает один из свободных участков памяти фиксированного размера, а когда запрашивается освобождение то, он просто сохраняет этот участок для дальнейшего использования.

Таким образом, распределение работает очень быстро, а фрагментация все ещё очень мала.

Теперь посмотрим работу этого аллокатора на примере отдав под управление аллокатора 12 байт (6 блоков, каждый по 2 байта).

Несколько основных составляющих пулового аллокатора:

1. `start` - указатель на начало аллокатора.
2. `end` - указатель на конец памяти аллокатора.
3. `freeblocks` - список из адресов свободных блоков в аллокаторе (сами звенья списка можно хранить в свободных блоках памяти, тем самым убрав дополнительные расходы с памятью).

![[pool-allocator-1 1.svg]]

Происходит запрос на выделение одного блока памяти. Аллокатор делает следующие вещи:

1. Проверяет, есть ли звенья в списке свободных блоков, если звеньев нет - память в аллокаторе кончилась.
2. Если есть хотя бы одно свободное звено, то аллокатор достает корневое или хвостовое звено списка (зависит от реализации, в данных примерах достает хвостовое значение) и отдает его адрес пользователю.

![[pool-allocator-2 1.svg]]

Когда приходит запрос на выделение нескольких блоков сразу - происходит та же ситуация, что и в прошлом шаге.

![[pool-allocator-3 1.svg]]

Если от клиента пришла команда на освобождения блока памяти, то аллокатор добавит адрес освободившегося блока в конец хвоста своего списка **FreeBlocks**.

Так же стоит помнить про такой крайний случай: клиент выдал адрес **0xEFAB12** на освобождение, а таким адресом памяти наш аллокатор не управляет. И возможна ситуация, что мы отдадим память, которая нам не подконтрольна (это приведет к _undefined behavior_ или если очень сильно повезет, то просто к краху программы). Для избегания данной проблемы и используются указатели `begin` и `end`, которые позволяют понять, не ошибся ли клиент с адресом во время запроса очистки памяти.

![[pool-allocator-4.svg]]

Стоит еще сказать про одну проблему, которую можно встретить при работе с данным аллокатором.

Клиент так же может прийти с запросом освободить совершенно любой адрес, находящийся в области памяти аллоактора, но не равный адресу какого-то используемого блока. Например адрес **0xFFAA07**.

![[pool-allocator-5.svg]]

Эта операция приведет к _Undefined Behavior_.

Поэтому если есть необходимость дополнительно проверять, все ли правильно делает клиент, то существуют такие инструменты остлеживания:

- Хранить так же и адреса занятых блоков, добавив еще один список.
- Проверять адреса на кратность размеру блока в аллокаторе.

---

## Стековый аллокатор:

Данный аллокатор, по сути своей, является эволюцией Линейного Аллокатора. Только в отличии от линейного аллокатора, стековый может очищать память фрагментами, а не сразу всю выделенную.

Для реализации стекового аллокатора требуются следующие части:

1. `header` - блок, сохраняющий в себе указатель на текущий адрес памяти и перемещаем его вперед или назад при выделении или деаллокации памяти.
2. `start` - начало памяти аллокатора.
3. `used` - конец аллоцированной памяти внутри аллокатора (так же это может быть не указатель, а числовое значение, которое будет представлять собой сдвиг по памяти от начала до конца аллоцированной памяти)
4. `end` - конец аллокатора или другими словами, общий объем памяти (так же это может быть не указатель, а числовое значение, которое будет представлять собой сдвиг по памяти от начала до конца аллокатора)

Рассмотрим так же пример с аллоцированной памятью в 14 байт.

![[stack-allocator-1.svg]]

От клиента приходит запрос на выделение памяти в два байта, тогда в нашем стековом аллокаторе будут происходить следующие действия:

- Первым делом, мы выделяем `header` в самом блока со свободным пространством. В блоке `header` мы будем хранить сведения о том, сколько было выделено байт (в нашем примере это 2 байта).
- Выделяем 2 байта.
- Смещаем указатель `used`.

![[stack-allocator-2 1.svg]]

Переменная `allocated` хранит значение выделенной памяти.

А теперь пользователь вносит запрос на выделение 6 байт памяти:

![[stack-allocator-3.svg]]

Дело дошло до очистки памяти. Например, пользователь хочет очистить выделенные 6 байт памяти. Это будет происходить так:

- От указателя, полученного от пользователя, на первый байт блока памяти мы отнимем фиксированный размер заголовка. (Так же стоит помнить, что пользователь может передать указатель с памятью не принадлежащим данному аллокатору и на этом шаге стоит сделать проверку - куда указывает указатель, находится ли он в диапазоне от `start` до `end`)
- Потом мы возьмем количество байт под очистку из `allocated` , сложим их с количеством байт `header` и от пользовательского указателя, полученного в прошлом шаге - деаллоцируем всю память.
- Осталось только сдвинуть указатель `used` на конец используемой памяти.

![[stack-allocator-4.svg]]

---

## Примитивный стандартный аллокатор:

Данная реализация аллокатора будет очень сильно схожа с классическим аллоактором из стандартной библиотеки _STL_.

В основе алгоритма лежит взаимодействие с участками (_chunks_). Эти участки имеют статичный размер и он обязан быть кратен 4, и так же если происходит какое-то выделение памяти, не кратное 4 (к примеру 3), оно должно быть выравнено до 4-х.

Основные части примитивного стандартного аллокатора будут следующие:

1. `start` - указатель на начальное значение памяти внутри аллокатора.
2. `end` - указатель на конечное значение памяти внутри аллокатора.
3. `maxblock` - указатель на максимальный блок памяти.
4. `freeblocks` - множество, в котором будут хранится заголовки свободных блоков.
5. `header` - заголовок, хранящий в себе информацию об проаллоцированном участке памяти. Его размер будет 4 байта. Если память еще не проаллоцированна `header` будет содержать в себе информацию о количестве оставшейся памяти. Так же `header` разделяет между собой разные участки проаллоцированной памяти внутри аллокатора.

![[simple-standard-allocator-1.svg]]

От клиента поступает запрос на аллоцирование памяти в 2 байта. Аллокатор должен будет проделать шаги:

- Проверить достаточно ли памяти в аллоакторе под, нужное пользователю, количество байт. Если размер аллоцируемой памяти меньше, чем размер свободного, под выделение, пространства от `end` до `maxBlock` (`sizeof(end - maxBlock)`) - это значит, что памяти под выделение достаточно. Так же помним, что нам еще 4 байта `header` надо записать.
- Если памяти достаточно, то мы отдаем адрес памяти следующий за заголовком (как это делается в стековом аллокаторе).
- Удаляем значение прошлого заголовка в контейнере `FreeBlocks`.
- Добавляем в контейнер `FreeBlocks` новое значение конца только что проаллоцированной памяти.
- Выставляем на конец проаллоцированной памяти `maxBlock`.
- Добавляем в конец проаллоцированной памяти разделяющий `header`.

![[simple-standard-allocator-2.svg]]

Теперь от клиента поступает запрос на аллоцирование 6 байт памяти. Происходят все шаги по аллокации, описанные ранее, но еще добавляются:

- Когда выделяется максимально возможное количество памяти `maxBlock == nullptr`.
- Из контейнера `FreeBlocks` удаляются все значения свободной памяти, относящиеся к данному Участку.

![[simple-standard-allocator-3.svg]]

В нашем единственном Участке аллокатора закончилась память. Теперь, для возможности выделения еще памяти аллокатор создаст второй участок, который будет пуст.

![[simple-standard-allocator-4.svg]]

Теперь у нас есть два Участка внутри аллокатора. Память в первом участке полностью забита. Память во втором участке свободна. И предположим клиент запросил 8 байт памяти:

![[simple-standard-allocator-4 1.svg]]

Вспомним наше правило, по поводу того, что каждый Участок должен быть кратен 4. Ответ очень просто – это делается для простоты реализации и восприятия алгоритма.

Так как возможна ситуация, что в конце Участка может остаться некоторая память, в которую не поместиться заголовок.

Для решения такой проблемы можно:

- Сделать выравнивание.
- Сделать заголовок меньшего размера
- Использование дополнительных средств для отслеживания ситуаций такого плана.

Это довольно важная проблема, которую стоит отслеживать, так как иначе такая память будет потеряна и самое главное, что эта потеря потенциально с каждым новым Участком может расти.

![[simple-standard-allocator-5.svg]]

В ситуации, когда клиент запрашивает освобождение какого-то блока происходят следующие действия:

1. Определить в каком участке (на картинке первый или второй) находится нужный, на удаление блок. Обычно это происходит за линейное время по суммарному количству участков в памяти.
2. Далее алгоритм очистки такой же, как и у стекового аллокатора.
3. Добаим адрем заголовка освобожденного блока во множество свободных блоков `FreeBlocks`.
4. Обновим указатель `MaxBlock` если размер только что освобожденного блока больше, чем `MaxBlock`.

![[simple-standard-allocator-6.svg]]

Важно понимать, что в данной реализации, при каждом последующем освобождении памяти происходит попытка дефрагментации в том участке из которого была освобождена память.

Дефрагментация нужна, что бы можно было объединять свободные блоки в блоки большего размера.

Например, если клиент запросит еще одну очистку памяти, но в этот раз по адресу `0xFFAA06`, первого участка, будет следующая картина:

![[simple-standard-allocator-7.svg]]

После запроса на очистку, у есть полностью свободный участок, но два `header` занимают память, которую занимать не должны, соответственно, нужно произвести дефрагментацию памяти.

Дефрагментация в данной реализациии - крайне примитивная операция. Суть в том, что сразу очистки того или иного блока памяти **сразу** идет проверка, не свободны ли два соседних блока слева и справа от освобожденного. И если два соседствующих блока свободны, то они объединяются в один цельный блок.

![[simple-standard-allocator-8.svg]]

- **Замечание.**
    
    Также стоит отметить, что данная реализация будет катастрофически плохо работать с выделением маленьких блоков памяти, например равных 1 байту. В такой ситуации мы получаем +7 лишних байт на выделение всего лишь одного байта памяти из-за того, что размер заголовка равен 4 байтам и еще 3 байта на выравнивание адресов, которые должны быть кратны четырем.
    
    Это сказано как напоминание, что не следует слепо использовать какой-либо алгоритм распределения памяти. Так как вместо оптимизации можно будет получить только дополнительные траты.
    
    ![[simple-standard-allocator-9.svg]]
    

### Реализация

---

Исходя из требований к аллокаторам в главе стандарта: "_Allocator requirements [allocator.requirements]_" примитивный интерфейс аллокатора, который может использоваться в _STL_, должен выглядеть примерно так:

```C++
template <class T>
class Allocator
{
		typedef T value_type;
		
		Allocator(some_args);
		
		template <class T> 
    Allocator(const <T>& other);

		T* allocate(std::size_t count_objects);

		void deallocate(T* ptr, std::size_t count objects);
};

template<class T, class U>
bool operator==(const Allocator<T>&, const Allocator<U>&);

template<class T, class U>
bool operator!=(const Allocator<T>&, const Allocator<U>&);
```

Предполагается, что _STL_ контейнеры обращаются к аллокатору не напрямую, а через шаблон `std::allocator_trait`, котый предоставляет следующие значения:

```C++
typedef T* pointer;
typedef const T* const_pointer;
```

После освещения основных требований, можно приступить к написанию интерфейса.

Для начала опишем прослойку, в которой при помощи стратегий (`AllocationStrategy`) мы сможем без проблем менять алгоритм аллоцирования памяти для определенных целей.

```C++
template<typename T, class AllocationStrategy>
class Allocator
{
    static_assert(!std::is_same_v<T, void>,
                  "Type of the allocator can not be void");

public:
    using value_type = T;

    template<typename U, class AllocStrategy>
    friend class Allocator;

    template<typename U>
    struct rebind
    {
        using other = Allocator<U, AllocationStrategy>;
    };

public:
    Allocator() = default;

    explicit Allocator(AllocationStrategy& strategy) noexcept
        : m_allocation_strategy{&strategy}
    {
    }

    Allocator(const Allocator& other) noexcept
        : m_allocation_strategy{other.m_allocation_strategy}
    {
    }

    template<typename U>
    Allocator(const Allocator<U, AllocationStrategy>& other) noexcept
        : m_allocation_strategy{other.m_allocation_strategy}
    {
    }

    T* allocate (std::size_t count_objects)
    {
        assert(m_allocation_strategy && "Not initialized allocation strategy");
        return static_cast<T*>(
            m_allocation_strategy->allocate(count_objects * sizeof(T)));
    }

    void deallocate (void* memory_ptr, std::size_t count_objects)
    {
        assert(m_allocation_strategy && "Not initialized allocation strategy");
        m_allocation_strategy->deallocate(memory_ptr,
                                          count_objects * sizeof(T));
    }

    template<typename U, typename... Args>
    void construct (U* ptr, Args&&... args)
    {
        new (reinterpret_cast<void*>(ptr)) U{std::forward<Args>(args)...};
    }

    template<typename U>
    void destroy (U* ptr)
    {
        ptr->~U();
    }

private:
    AllocationStrategy* m_allocation_strategy = nullptr;
};

template<typename T, typename U, class AllocationStrategy>
bool operator==(const Allocator<T, AllocationStrategy>& lhs,
                const Allocator<U, AllocationStrategy>& rhs)
{
    return lhs.m_allocation_strategy == rhs.m_allocation_strategy;
}

template<typename T, typename U, class AllocationStrategy>
bool operator!=(const Allocator<T, AllocationStrategy>& lhs,
                const Allocator<U, AllocationStrategy>& rhs)
{
    return !(lhs == rhs);
}
```

И благодаря стратегиям для распределения памяти, мы можем писать примерно следующим образом:

```C++
template<typename T> 
using AllocatorForSmallObjects = Allocator<T, StrategyForSmallObjects>;

template<typename T>
using AllocatorForBigObjects = Allocator<T, StrategyForBigObjects>;
```

Такой подход дает нам право гибко менять алгоритмы распределения памяти в той или иной ситуации. Единственное и основное требование к `AllocationStrategy` - у нее обязаны быть методы `allocate` и `deallocate`.

```C++
namespace details
{

    std::size_t getAlignmentPadding(std::size_t not_aligned_address, std::size_t alignment)
    {
        if ( (alignment != 0u) && (not_aligned_address % alignment != 0u) )
        {
            const std::size_t multiplier = (not_aligned_address / alignment) + 1u;
            const std::size_t aligned_address = multiplier * alignment;
            return aligned_address - not_aligned_address;
        }

        return 0u;
    }

    // Current chunk implementation works only with size
    // aligned by 4 bytes, because HEADER_SIZE now also 4 bytes.
    // You can modify it with HEADER_SIZE without problems for your purposes.

    template<std::size_t CHUNK_SIZE>
    class Chunk
    {
        static constexpr std::size_t HEADER_SIZE = 4u;
        static_assert(CHUNK_SIZE % HEADER_SIZE == 0, "CHUNK_SIZE must be multiple of the four");
        static_assert(CHUNK_SIZE > HEADER_SIZE, "CHUNK_SIZE must be more than HEADER_SIZE");
    public:
        Chunk()
        {
            m_blocks.resize(CHUNK_SIZE);
            std::uint32_t* init_header = reinterpret_cast<std::uint32_t*>(m_blocks.data());
            *init_header = CHUNK_SIZE - HEADER_SIZE;
            m_max_block = init_header;
            m_free_blocks.insert(init_header);
        }
        
        bool isInside(const std::uint8_t* address) const noexcept
        {
            const std::uint8_t* start_chunk_address = reinterpret_cast<const std::uint8_t*>(m_blocks.data());
            const std::uint8_t* end_chunk_address = start_chunk_address + CHUNK_SIZE;
            return (start_chunk_address <= address) && (address <= end_chunk_address);
        }
        
        std::uint8_t* tryReserveBlock(std::size_t allocation_size)
        {
            const std::size_t not_aligned_address = reinterpret_cast<std::size_t>(m_max_block) + allocation_size;
            const std::size_t alignment_padding = getAlignmentPadding(not_aligned_address, HEADER_SIZE);
            const std::uint32_t allocation_size_with_alignment = static_cast<std::uint32_t>(allocation_size + alignment_padding);
            if ( (!m_max_block) || (allocation_size_with_alignment > *m_max_block) ) // Check on enaught memory for allocation
            {
                return nullptr;
            }
            
            // Find min available by size memory block
            const auto min_it = std::min_element(m_free_blocks.cbegin(), m_free_blocks.cend(), [allocation_size_with_alignment] (const std::uint32_t* lhs, const std::uint32_t* rhs)
            {
                if (*rhs < allocation_size_with_alignment)
                {
                    return true;
                }
                
                return (*lhs < *rhs) && (*lhs >= allocation_size_with_alignment);
            });
            
            assert(min_it != m_free_blocks.cend() && "Internal logic error with reserve block, something wrong in implementation...");
            assert(**min_it >= allocation_size_with_alignment && "Internal logic error with reserve block, something wrong in implementation...");
            
            std::uint32_t* header_address = *min_it;
            std::uint32_t* new_header_address =
                reinterpret_cast<std::uint32_t*>(reinterpret_cast<std::uint8_t*>(header_address) + HEADER_SIZE + allocation_size_with_alignment);
            if (m_free_blocks.find(new_header_address) == m_free_blocks.cend()) // check if there is free memory in the current block
            {
                const std::uint32_t old_block_size = *header_address;
                const std::uint32_t difference = old_block_size - HEADER_SIZE;
                if (difference >= allocation_size_with_alignment) // check if there is enough space for another block
                {
                    const std::uint32_t new_block_size = difference - allocation_size_with_alignment;
                    *new_header_address = new_block_size;
                    m_free_blocks.insert(new_header_address);
                }
            }
            
            m_free_blocks.erase(header_address);
            *header_address = static_cast<std::uint32_t>(allocation_size);
            if (header_address == m_max_block) // if the maximum block were changed, then need to find the maximum block again
            {
                // Find max block by size
                const auto max_it = std::max_element(m_free_blocks.cbegin(), m_free_blocks.cend(), [] (const std::uint32_t* lhs, const std::uint32_t* rhs)
                {
                    return (*lhs) < (*rhs);
                });
                
                // If there are no free blocks, therefore the memory in this chunk is over
                m_max_block = (max_it != m_free_blocks.cend()) ? (*max_it) : (nullptr);
            }

            return reinterpret_cast<std::uint8_t*>(header_address) + HEADER_SIZE;
        }
        
        void releaseBlock(std::uint8_t* block_ptr)
        {
            std::uint8_t* header_address = block_ptr - HEADER_SIZE;
            const std::uint32_t size_relized_block = *header_address;
            if ( (!m_max_block) || (size_relized_block > *m_max_block) ) // if the relized block is greater than the maximum, then need to replace it
            {
                m_max_block = reinterpret_cast<std::uint32_t*>(header_address);
            }
                
            m_free_blocks.insert(reinterpret_cast<std::uint32_t*>(header_address));
            auto forward_it = m_free_blocks.find(reinterpret_cast<std::uint32_t*>(header_address));
            auto backward_it = tryDefragment(forward_it, m_free_blocks.end());
            tryDefragment(std::make_reverse_iterator(backward_it), m_free_blocks.rend());
        }
    private:
        template<typename DstIterator, typename SrcIterator>
        constexpr DstIterator getIterator(SrcIterator it) const
        {
            using iterator = std::set<std::uint32_t*>::iterator;
            using reverse_iterator = std::set<std::uint32_t*>::reverse_iterator;
            if constexpr ( (std::is_same_v<SrcIterator, iterator>) && (std::is_same_v<DstIterator, reverse_iterator>) )
            {
                return std::make_reverse_iterator(it);
            }
            else if constexpr ( (std::is_same_v<SrcIterator, reverse_iterator>) && (std::is_same_v<DstIterator, iterator>) )
            {
                return it.base();
            }
            else
            {
                return it;
            }
        }
        
        template<typename Iterator>
        Iterator tryDefragment(Iterator start_it, Iterator end_it)
        {
            // primitive defragmentation algorithm - connects two neighboring
            // free blocks into one with linear complexity
   
            auto current_it = start_it++;
            auto next_it = start_it;
            std::uint32_t* current_header_address = *current_it;
            if ( (current_it != end_it) && (next_it != end_it) )
            {
                std::uint32_t* next_header_address = *next_it;
                const std::uint32_t current_block_size = *current_header_address;
                const std::uint32_t* available_current_block_address =
                    reinterpret_cast<std::uint32_t*>(reinterpret_cast<std::uint8_t*>(current_header_address) + HEADER_SIZE + current_block_size);
                if (available_current_block_address == next_header_address)
                {
                    const std::uint32_t next_block_size = *next_header_address;
                    const std::uint32_t new_current_block_size = current_block_size + HEADER_SIZE + next_block_size;
                    *current_header_address = new_current_block_size;
                    if (new_current_block_size > *m_max_block)
                    {
                        m_max_block = reinterpret_cast<std::uint32_t*>(current_header_address);
                    }
                            
                    auto delete_it = getIterator<std::set<std::uint32_t*>::iterator>(next_it);
                    return getIterator<Iterator>(m_free_blocks.erase(delete_it));
                }
            }
            
            return current_it;
        }
    public:
        std::vector<std::uint8_t*> m_blocks;
        std::set<std::uint32_t*> m_free_blocks;
        std::uint32_t* m_max_block;
    };
```

Теперь немного о том, как можно украсить использование аллокаторов вместе со стандартными контейнерами:

```C++
template<typename T, std::size_t CHUNK_SIZE = 16'384u>
using CustomAllocator = Allocator<T, CustomAllocationStrategy<CHUNK_SIZE>>;

template<typename T>
using CustomAllocatorWithHeapChunks
    = Allocator<T, CustomAllocationStrategy<16'384u>>;

template<typename T>
using custom_vector = std::vector<T, CustomAllocator<T>>;

template<typename T>
using custom_list = std::list<T, CustomAllocator<T>>;

template<typename T>
using custom_set = std::set<T, std::less<T>, CustomAllocator<T>>;

template<typename T>
using custom_unordered_set
    = std::unordered_set<T, std::hash<T>, std::equal_to<T>, CustomAllocator<T>>;

template<typename K, typename V>
using custom_map
    = std::map<K, V, std::less<K>, CustomAllocator<std::pair<const K, V>>>;

template<typename K, typename V>
using custom_unordered_map
    = std::unordered_map<K,
                         std::hash<K>,
                         std::equal_to<K>,
                         CustomAllocator<std::pair<const K, V>>>;

using custom_string
    = std::basic_string<char, std::char_traits<char>, CustomAllocator<char>>;
```

Можно также использовать аллокаторы и с умными указателями, но для этого придется написать небольшую прослойку:

```C++
template<typename T>
using custom_unique_ptr = std::unique_ptr<T, std::function<void(T*)>>;

template<class Allocator, typename T = typename Allocator::value_type, typename... Args>
custom_unique_ptr<T> make_custom_unique(Allocator allocator, Args&&... args)
{
    const auto custom_deleter = [allocator](T* ptr) mutable
    {
        allocator.destroy(ptr);
        allocator.deallocate(ptr, 1u);
    };
        
    void* memory_block = allocator.allocate(1u);
    if (memory_block)
    {
        T* object_block = static_cast<T*>(memory_block);
        allocator.construct(object_block, std::forward<Args>(args)...);
        return custom_unique_ptr<T>{ object_block, custom_deleter };
    }

    return nullptr;
}
```

Ну и теперь, наконец, пример использования всего этого:

```C++
int main(int argc, char** argv)
{
    CustomAllocationStrategy allocation_area{};

    CustomAllocator<int> custom_int_allocator{ allocation_area };
    custom_vector<int> vector{ custom_int_allocator };
    for (int i = 0u; i < 100; ++i)
    {
        vector.push_back(i);
        std::cout << vector.at(i) << " ";
    }
    
    vector.resize(16u);
    for (int val : vector)
    {
        std::cout << val << " ";
    }

CustomAllocator<int> custom_int_allocator_copy = vector.get_allocator();
    custom_unique_ptr<int> ptr1 = make_custom_unique<CustomAllocator<int>>(custom_int_allocator_copy, 100);
    custom_unique_ptr<int> ptr2 = make_custom_unique<CustomAllocator<int>>(custom_int_allocator_copy, 500);
    custom_unique_ptr<int> ptr3 = make_custom_unique<CustomAllocator<int>>(custom_int_allocator_copy, 1000);
    custom_unique_ptr<int> ptr4 = make_custom_unique<CustomAllocator<int>>(custom_int_allocator_copy, 1500);
    std::cout << *ptr1 << " " << *ptr2 << " " << *ptr3 << " " << *ptr4 << " ";
    
    CustomAllocator<float> custom_float_allocator { custom_int_allocator };
    custom_list<float> list{ { 10.0f, 11.0f, 12.0f, 13.0f, 14.0f, 15.0f }, custom_float_allocator };
    for (float val : list)
    {
        std::cout << val << " ";
    }
    
    CustomAllocator<std::pair<double, double>> custom_pair_allocator{ allocation_area };
    custom_map<double, double> map{ { { 1.0, 100.0 }, { 2.0, 200.0 } }, custom_pair_allocator };
    for (const auto& it : map)
    {
        std::cout << "{" << it.first << " : " << it.second << "} ";
    }
    
    CustomAllocator<double> custom_double_allocator{ allocation_area };
    custom_set<double> set{ { 1000.0, 2000.0, 3000.0 }, custom_double_allocator };
    for (double val : set)
    {
        std::cout << val << " ";
    }

    CustomAllocator<char> custom_char_allocator{ allocation_area };
    custom_string string1{ "First allocated string without SBO ", custom_char_allocator };
    custom_string string2{ "Second allocated string without SBO ", custom_char_allocator };
    custom_string string3{ "Third allocated string without SBO ", custom_char_allocator };
    custom_string result_string = string1 + string2 + string3;
    std::cout << result_string;
    
    return EXIT_SUCCESS;
}
```