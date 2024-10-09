---

## Вопрос первый.

_Вопрос:_

Нормально ли выделять в куче массив с нулем элементов?

```C++
\#include <iostream>

int main()
{       
    int* array = new int[0];
    delete[] array;

    return EXIT_SUCCESS;
}
```

_Ответ:_

Да, это корректно, но инструкция не имеет смысла.

Как вообще работает инструкция `new` . После ее вызова происходит две вещи:

1. Выделение памяти. Вызывается `void *malloc(``[size_t](https://en.cppreference.com/w/c/types/size_t)` `size);`.
2. Вызов конструктора объекта, под кого выделяется память.

Так же стоит помнить, что в сам `malloc(bytes)` передается количество байт, которые нужно выделить. Отсюда следует вопрос, а как тогда `void free(void *ptr);` может знать сколько байт удалять, если передается только указатель?

`void *malloc(``[size_t](https://en.cppreference.com/w/c/types/size_t)` `size);` - работает следующим образом, если малок выделяет какую-то память, он начинает указывать на начало, потом от этого начала отступает на один или два байта, и там сохраняет _Header_, в котором будет хранится информация о том, сколько памяти было выделено:

![[malloc.svg]]

И соответственно у `void free(void *ptr);` не возникает проблем с удалением всей памяти, потому что в него передается указатель на первый элемент, от этого указателя отступаем, считываем данные о количестве выделенной памяти из _Header_. И удалем весь массив:

![[free.svg]]

И отвечая на поставленный выше вопрос: `int* array = new int[0];` - аллокация нуля элементов массива бесполезна, потому что проаллоцируется только _Header_.

---

## Вопрос второй.

_Вопрос:_

В чем разница, между `new` и `malloc()`?

1. `malloc()` просто выделяет память, а `new` ещё и вызывает конструктор.
2. Если не вышло выделение памяти `malloc()` вернет `nullptr`, а `new` вернет `std::bad_alloc`.
    - _**Важное уточнение:**_
        
        Можно использовать **type dispatching** и перегрузить `new` при помощи `std::nothrow` и тогда в случае неудачи, он тоже вернет `nullptr`.
        
        ```C++
        \#include <iostream>
        \#include <new>
        
        int main()
        {       
            try 
            {   
                while (true)
                {
                    new int[1000'000'000ul];
                }
            }
            catch(const std::bad_alloc& e)
            {
                std::cout << e.what() << std::endl;
            }
        
            while (true)
            {
                int* p = new(std::nothrow) int[1000'000'000ul];
                if (p == nullptr)
                {
                    std::cout << "Allocation returned nullptr\n";
                    break;
                }
            }
        
            return EXIT_SUCCESS;
        }
        ```
        

---

## Вопрос третий.

_Вопрос:_

Почему `new` работает медленно и почему максимально стоит избегать выделения памяти из _heap_?

Есть 4 основных случая, почему стоит избегать частого выделения и очень сильно следить за этим:

1. Может произойти фрагментация _heap_.
2. При вызове `new` происходят звонки в ядро, то есть всякие `syscall`, а это всегда медленно.
3. `malloc()` - это _general purpose allocator_, как написано в стандарте, это значит, что он для 4 байт будет выделять память, как для 4 гигабайт. Никакие оптимизации не предусмотрены.
4. Если в линуксе прилинковаться к _pthread_, то `malloc()` будет потокобезопасным, хотя возможно это было и ненужно.

---

## Вопрос четвертый.

```C++
\#include <iostream>
\#include <new>

void handler()
{
    std::cout << "allocation failed" << std::endl;
    std::set_new_handler(nullptr);
}

int main()
{
    size_t allocations_count = 0u;
    std::set_new_handler(handler);

    try
    {
        while (true)
        {
            constexpr size_t GB = 1024u * 1024u * 1024u;
            new char[GB]; // Здесь происходит swapping

            std::cout << "allocated " << ++allocations_count << " GB" << std::endl;
        }
    }
    catch (const std::bad_alloc& e)
    {
        std::cout << e.what() << '\n';
    }

    std::cout << "finally allocated " << allocations_count << " GB" << std::endl;
}
```

Вопрос следующий, как вообще поведет себя система? Сколько выйдет проаллоцировать памяти?

Вывод будет таким:

```Bash
allocated 1 GB
...
allocated 131059 GB
allocated 131060 GB
allocated 131061 GB
allocated 131062 GB
allocation failed
std::bad_alloc
finally allocated 131062 GB
```

А как вообще система могла выделить так много памяти, если не в _RAM_, не на жестком диске нет столько гигабайт.

При аллокации памяти, память аллоцируется _постранично_, используя концепцию **виртуальной памяти**. Виртуальная память позволяет обращаться к большему количеству памяти, чем есть физически в _RAM_. Как это работает? _Ram_ разбивается на определенные сегменты, как раз их и называют **страницами** (_размер одной виртуальной страницы чаще всего 4 киллобайта = 4096 байт_). И приложение общается не с физической памятью, а с виртуальными адресами, которые только потом стыкуются с физическими.

Где это применяется?

Приложение использует память, проаллоцировав одну, две, три страницы. И физическая _RAM_ память закончилась. Что делать в таком случае? Страницы, которые дольше всего не использовались выгрузит на жесткий диск, а в освободившееся место начнет писать новые данные. Но если понадобятся данные из старых страниц, эти данные также выгрузятся с диска и затрут самые, давно используемые, страницы. И поэтому этот процесс называется **swapping**.

Таким образом у нас происходит **swapping** в месте аллокации `new char[GB];` , память то выделяется, но при этом данные у нас не изменяются, а значит какой смысл загружать их на жесткий диск? Вот они и крутятся в _RAM_ постоянно перезаписывая страницы друг друга.

Добавим изменение этих данных:

```C++
\#include <iostream>
\#include <new>

void handler()
{
    std::cout << "allocation failed" << std::endl;
    std::set_new_handler(nullptr);
}

int main()
{
    size_t allocations_count = 0u;
    std::set_new_handler(handler);

    try
    {
        while (true)
        {
            constexpr size_t GB = 1024u * 1024u * 1024u;

            char* buffer = new char[GB];
            for (size_t i = 0u; i < GB; i += 4096)
            {
                buffer[i] = 100;
            }

            std::cout << "allocated " << ++allocations_count << " GB" << std::endl;
        }
    }
    catch (const std::bad_alloc& e)
    {
        std::cout << e.what() << '\n';
    }

    std::cout << "finally allocated " << allocations_count << " GB" << std::endl;
}
```

Вывод будет таким:

```Bash
allocated 1 GB
...
allocated 28 GB
killed
```

И происходит это из-за изменения данных, и записи их на жесткий диск.

---

## Вопрос пятый.

Есть следующий код:

```C++
\#include <iostream>

struct Data
{
    Data()
    {
        throw std::runtime_error("exception");
    }
};

int main()
{
    try
    {
        Data* data = new Data;
        delete data;
    }
    catch (const std::exceptions& e)
    {
    }

    return EXIT_SUCCESS;
}
```

_Вопрос_:

Будет ли здесь утечка памяти?

Вспомним, что `new` отрабатывает в два этапа, и это:

1. Сначала вызывается `malloc()`.
2. Потом вызывается конструктор `Data()`.

Тут исключение происходит в конструкторе, а это значит, что `malloc()` успел выделить память. Но стоит вспомнить, что действия `new` происходят транзакционно (`new` соблюдает строгую гарантию исключений по стандарту) и память, которую выделил `malloc()`, **будет очищена**.

---

## Вопрос шестой.

Есть следующий код:

```C++
\#include <iostream>

struct Data
{
    Data()
    {
        std::cout << "Data" << std::endl;
    }
    ~Data()
    {
        std::cout << "~Data" << std::endl;
    }
};

char buffer[1024] = { 0 };

int main()
{
    Data* data_1 = new (buffer) Data; // placement new
    Data* data_2 = new (buffer + sizeof(Data)) Data;
    Data* data_3 = new (buffer + sizeof(Data) * 2) Data;

    return EXIT_SUCCESS;
}
```

Обыкновенный `new` выделяет память в где-то рандомно _Heap_, а существует такой механизм как **placement new**, который помогает выделять память в каком-то конкретном буфере (буфер, к примеру может находится на стеке). Другими словами **placement new** позволяет передать адрес, где должен сконструироваться объект через вызов `new`.

_Вопрос:_

Все ли нормально с этим кодом?

Если его скомпилировать, то получится следующий вывод:

```C++
Data
Data
Data
```

То есть деструктор не вызывается.

Но как нам правильно удалить данные объекты? Вызвать `delete data_1`, `delete data_2`, `delete data_3` ?

**Нет, так мы получим только UB.**

Решить проблему можно только так:

```C++
\#include <iostream>

struct Data
{
    Data()
    {
        std::cout << "Data" << std::endl;
    }
    ~Data()
    {
        std::cout << "~Data" << std::endl;
    }
};

int main()
{
    char buffer[1024] = { 0 };

    Data* data_1 = new (buffer) Data;
    Data* data_2 = new (buffer + sizeof(Data)) Data;
    Data* data_3 = new (buffer + sizeof(Data) * 2) Data;

    data_1->~Data(); // Явный вызов деструктора.
    data_2->~Data();
    data_3->~Data();

    return EXIT_SUCCESS;
}
```