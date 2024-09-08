---

### Ресурсы:

1. [[link](https://www.cprogramming.com/tutorial/size_of_class_object.html)] Size of class.
2. [[link](https://www.geeksforgeeks.org/why-is-the-size-of-an-empty-class-not-zero-in-c/)] If nothing in class - what size.

---

## Что такое размер класса и как его определить:

_Размер класса_ - это количество байт в памяти, которые занимает созданный объект какого-то класса.

Размер класса в языке `C++` можно определить двумя способами:

1. Вызвать оператор `sizeof` , способ работает только с полностью созданными объектами.
2. Вызвать функцию `sizeof()` , способ отличается тем, что в функцию можно передать не только полностью готовый объект, но и его тип.

Пример использования:

```C++
class Interface
{
public:
    Interface() = default;

private:
    int32_t fieldOne;
};

int main()
{       
    Interface iface;
    std::cout << "bytes: " << sizeof iface      << std::endl;
    std::cout << "bytes: " << sizeof(Interface) << std::endl;
}
```

```Bash
bytes: 4
bytes: 4
```

---

## Факторы влияющие на размер класса:

Перечислим основные факторы, которые будут влиять на количество, занимаемых в памяти, байт объектом какого-то класса:

1. [[C++. Сколько будут весить различные классы в С++]]. Сколько места в памяти занимает тип такого поля, столько мы мы и прибавляем к общему размеру нашего класса.
2. Важен так же [[C++. Сколько будут весить различные классы в С++]], так как существует такая вещь, как [выравнивание](https://ru.stackoverflow.com/questions/435726/%D0%92%D1%8B%D1%80%D0%B0%D0%B2%D0%BD%D0%B8%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B4%D0%B0%D0%BD%D0%BD%D1%8B%D1%85). [[TODO-]] Сделать конспект по выравниваниям.
3. Собственно, эти самые [[C++. Сколько будут весить различные классы в С++]] и заполнения байтов можно вынести в отдельный пункт.
4. [[C++. Сколько будут весить различные классы в С++]].
5. [[C++. Сколько будут весить различные классы в С++]] (динамический полиморфизм, использующий виртуальные функции).
6. [[C++. Сколько будут весить различные классы в С++]], сколько байт он присваивает тем или иным классическим типам данных.
7. [[C++. Сколько будут весить различные классы в С++]] (имеется в виду присутствие виртуального наследования).

Теперь разберем каждый пункт и приведем примеры.

---

### 1. Размер не статических полей класса:

Если мы хотим найти _размер класса_, где есть только поля класса - то статические поля не будут считаться, как часть размера класса.

```C++
class Interface
{
public:
    Interface() = default;

private: 
        char fieldOne{0};            // 1 byte -> 4 bytes by align  
        float fieldTwo{0};           // 4 bytes
        const int32_t fieldThree{0}; // 4 bytes

        static int32_t fieldFour; // 0 bytes
        static int32_t fieldFive; // 0 bytes
};

int main()
{       
    Interface iface;
    std::cout << "bytes: " << sizeof iface << std::endl;
}
```

```Bash
bytes: 12
```

Другими словами для объекта класса `Interface` размер будет считаться так:

`sizeof fieldOne + sizeof fieldTwo + sizeof fieldThree`

Статические поля не то, что бы часть объекта, они лишь ассоциированы с объектом.

---

### 2. Порядок определения полей в классе:

```C++
class Interface
{
public:
    Interface() = default;

private: 
		char char1; 
    int int1; 
    int int2; 
    int int3; 
    long longInt; 
    short shortInt;
};

int main()
{       
    Interface iface;
    std::cout << "bytes: " << sizeof iface      << std::endl;
    std::cout << "bytes: " << sizeof(Interface) << std::endl;
}
```

```Bash
bytes: 24
bytes: 24
```

Размер класса будет 24 байта, хоть `char` и занимает всего лишь 1 байт, рядом с ним в памяти будет добавлено еще невидимых байта. Это происходит потому, что следующий после `char` это тип `int` (он занимает в памяти 4 байта).

[[TODO-]] Дополнить информацию по align.

Если же переписать класс следующим образом, то класс уже будет весить 20 байт:

```C++
class Interface
{
public:
    Interface() = default;

private: 
    int int1; 
    int int2; 
    int int3; 
    long longInt; 
    short shortInt;
		char char1; 
};

int main()
{       
    Interface iface;
    std::cout << "bytes: " << sizeof iface      << std::endl;
    std::cout << "bytes: " << sizeof(Interface) << std::endl;
}
```

```Bash
bytes: 20
bytes: 20
```

---

### 3. Выравнивания:

Дописать этот пункт, когда будут готовы TODO выше.

---

### 4. Зависимость от размера базового класса:

В памяти, когда один класс (наследник) наследуется от другого (родитель). То класс-наследник имеет внутри себя все поля и методы класса-родителя:

```C++
class Interface
{
    int int1; 
};

class Derived : public Interface
{ 
    int int2; 
    int int3; 
};

int main()
{       
    Interface iface;
    Derived   der;
    std::cout << "bytes Interface: " << sizeof iface  << std::endl;
    std::cout << "bytes Derived  : " << sizeof der    << std::endl;
}
```

```Bash
bytes Interface: 4
bytes Derived  : 12
```

---

### 5. Существование виртуальных функций:

Существование виртуальной функции в классе добавляет дополнительные 4 байта (или 8, или 12) к размеру класса. Потому что в классе появляется указатель на [[С++. Что такое виртуальная таблица методов]].

Так же стоит отметить, что если класс-родитель уже имеет виртуальную функцию, и класс-наследник имеет виртуальную функцию, то указатель на виртуальную таблицу будет только один. Этот указатель будет гулять по всей возможной иерархии в единственном числе.

```C++
class Interface
{
public:
    virtual void virt() {}; // 12

private:
    int int1;               // 4
};

class Derived : public Interface
{ 
public:
    virtual void virt() {}; // 0 

private:
    int int2;               // 4
    int int3;               // 4
};

int main()
{       
    Interface iface;
    Derived   der;
    std::cout << "bytes Interface: " << sizeof iface  << std::endl;
    std::cout << "bytes Derived  : " << sizeof der    << std::endl;
}
```

```Bash
bytes Interface: 16
bytes Derived  : 24
```

То есть математика простая:

`sizeof Interface = sizeof PointerVTable + sizeof int = 12 + 4 = 16`.

`sizeof Derived = sizeof Interface + sizeof int + sizeof int = 16 + 4 + 4 = 24`.

---

### 6. Использование разных компиляторов:

В определенных сценариях размер класса объекта будет скомпилирован специфическим образом.

```C++
class Interface
{ 
        int int1; 
        char char1; 
}; 
 
class Derived : public Interface{ 
        char char2; 
        int int2; 
};

int main()
{       
    Derived   der;
    std::cout << "bytes Derived  : " << sizeof der    << std::endl;
}
```

Код выше, при использовании msvc скомпилируется так:

```Bash
bytes Interface: 16
```

А с компилятором gcc скомпилируется так:

```Bash
bytes Interface: 12
```

Это происходит из-за разного подхода к циклу чтения/записи памяти.

[[TODO-]] Прочитать про memory read/write cycle и сделать здесь ссылку.

---

### 7. Режим наследования:

Иногда в `C++` приходится использовать виртуальное наследование (один из классических примеров - реализация финального класса). Когда мы используем виртуальное наследование, накладные расходы на указатель виртуального базового класса в этом классе составят 4 байта (или в зависимости от компилятора).

```C++
class Interface
{
    int int1;
};

class A_Derived : public virtual Interface
{ 
    int int2; 
};

class B_Derived : public virtual Interface
{ 
    int int3;
};

class ABC_Derived : public A_Derived, public B_Derived { 
    int int5; 
};
```

```Bash
bytes Interface   : 4
bytes A_Derived   : 16
bytes B_Derived   : 16
bytes ABC_Derived : 40
```

`sizeof Interface = sizeof int = 4`

`sizeof A_Derived = sizeof int + Interface + sizeof PointerVTable = 4 + 4 + 8 = 16`

`sizeof B_Derived = sizeof int + Interface + sizeof PointerVTable = 4 + 4 + 8 = 16`

`sizeof ABC_Derived = sizeof int + A_Derived + B_Derived - sizeof PointerVTable = 4 + 16 + 16 + 12 - 8` (12 и 8 - разные указатели на `VTable`).

  

Так же стоит отдельно упомянуть что

```Bash
class Interface
{
};

int main ()
{       
    Interface iface;

    std::cout << "bytes Interface   : " << sizeof iface  << std::endl;
}
```

Будет просто хранить один байт для того, что бы класс было видно в памяти (из-за уникального идентификатора памяти, по которому можно обращаться к классу):

```Bash
bytes Interface : 1
```