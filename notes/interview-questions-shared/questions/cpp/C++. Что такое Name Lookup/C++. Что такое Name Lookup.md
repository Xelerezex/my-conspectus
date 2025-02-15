Это процедура, когда имя функции, встретившееся в программе, **ассоциируется** с _объявлением функции_.

К примеру, в нашей программе встретилось выражение:

```C++
std::cout << std::endl;
```

Компилятор предпримет следующие действия:

1. Произойдет “поиск неполного имени” для имени _std_, и найдет объявление пространстве имён _std_ в хедере `<iostream>`.
2. Произойдет “поиск полного имени” для имени `cout`, и найдет объявление переменной _cout_ в пространстве имён `std`.
3. Произойдет “поиск полного имени” для имени `endl`, и найдет объявление шаблона функций в пространстве имён `std`.
4. Сначала произойдет “поиск зависящий от аргументов” для имени `operator<<`, который найдет несколько объявлений шаблонов функций в пространстве имен `std`, затем произойдет второй “поиск полного имени” для имени `std::ostream::operator<<`, который найдет несколько объявлений методов в классе `std::ostream`.

Для функций и шаблонов-функций “поиск имени” может связать несколько объявлений с одним именем. Еще и “Поиск зависящий от аргументов” может добавить к этой ассоциации несколько объявлений.

Для всех остальных имен (переменные, пространств имен, классов, и т.д.), “поиск имени” может предоставить несколько объявлений, только если они объявляют одну и ту же сущность.

### Типы “Поиска имени”:

1. Поиск полного имени
    
    [[C++. Что такое Qualified name lookup|C++. Что такое Qualified name lookup]]
    
2. Поиск неполного имени
    
    [[С++. Что такое Unqualified name lookup|С++. Что такое Unqualified name lookup]]
    
3. Поиск неполного имени, но для функций добавляется “поиск зависящий от аргументов”.
    
    [[C++. Что такое ADL|C++. Что такое ADL]]