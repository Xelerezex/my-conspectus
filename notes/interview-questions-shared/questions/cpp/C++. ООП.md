
---

## Вопрос первый.

_Вопрос:_

Что происходит в данном коде?

```C++
#include <iostream>

struct Int
{
	Int& operator ++ ()
	{
		++m_value;
		return *this;
	}

private:
	
	int m_value;

};

int main(int argc, char** argv)
{
	Int int_variable;
	++int_variable;
	int_variable++;
	++int_variable++;
	return EXIT_SUCCESS;
}
```

_Вопрос:_

Данный код упадет, потому что в нем нет постфиксного инкремента, и есть только переопределение префиксного.

Что бы сделать перегрузку постфиксного инкремента, надо сделать следующее:

```C++
Int& operator ++ (int)
{
	Int temp(*this); // Происходит копирование элемента
	++m_value;
	return temp;
}
```

Соответственно, префиксный инкремент будет работать быстрее, потому что не будет создаваться копия.

При этом стоит помнить, что если бы переменная `int_variable` была бы типа `int` и выполнялась бы следующая строка: `++int_variable++;` - то при компиляции была бы ошибка `lvalue required as increment operand`.

---

## Вопрос второй.

_Вопрос:_

Что в данном коде может пойти не так?

```C++
#include <iostream>

template<typename T>
struct Data
{
	explicit Data(T* ptr)
		: m_ptr{ptr} {}

	~Data()
	{
		delete m_ptr;
	}

private:

	T* m_ptr;

};

int main(int argc, char** argv)
{
	Data<int> data_1{new int{10}};
	Data<int> data_2 = data_1;
	return EXIT_SUCCESS;
}
```

В данном коде происходит поверхностное копирование. И предположим, что `new int{10}` в строке `Data<int> data_1{new int{10}};` теперь указывает на `0x00AA00`.

При поверхностном копировании в строке `Data<int> data_2 = data_1;` произойдет просто копирование указателя, а это значит, что и `data_2` и `data_1` указывают теперь на `0x00AA00`. И соответственно, при выходе из логического блока у данных объектов вызовется деструктор, а так как они оба указывают на одну и ту же память, то произойдет двойное удаление.

_Вопрос:_

Как здесь можно избежать двойного копирования?

Воспользоваться правилом [[С++. Что такое The Rule of Three|С++. Что такое The Rule of Three]]. И это значит, что если определен деструктор, то надо определить и оператор копирования и оператор присваивания.

---
## Вопрос третий.

_Вопрос:_

Что в данном коде может пойти не так?

```C++
#include <iostream>

struct Data
{
	explicit Data(T* ptr)
		: m_size{size}, m_buffer{new int[m_size]} {}

	~Data()
	{
		delete[] m_buffer;
	}

private:

	int* m_buffer;
	size_t m_size;

};

int main(int argc, char** argv)
{
	Data data{10u};
	return EXIT_SUCCESS;
}
```

В данном коде будет **UB**, потому что первым всегда будет инициализироваться переменная, которая объявлена раньше, то есть `m_buffer` . А так как она будет первая проинициализирована, то в `m_size` может находится любой мусор.

---
## Вопрос четвертый.

_Вопрос:_

Что выведет данный код?

```C++
#include <iostream>

struct Base {};
struct Derived : Base {};

int main(int argc, char** argv)
{
	std::cout << "sizeof Base:    " << sizeof(Base) << std::endl;
	std::cout << "sizeof Derived: " << sizeof(Derived) << std::endl;

	return EXIT_SUCCESS;
}
```

Код выведет:

```Bash
sizeof Base:    1
sizeof Derived: 1
```

Данная концепция называется _**EBO**_ (_Empty Base Optimization_) и она гласит, что пустой класс не может иметь нулевой размер. Должен быть минимальный размер в один байт, для того, что бы можно было обратиться к нему по указателю: `Base* b_ptr = &b;`

---
## Вопрос пятый.

_Вопрос:_

Что выведет данный код?

```C++
#include <iostream>

struct Base
{
	Base()
	{
		print();
	}

	void print()
	{
		std::cout << "Base::print" << std::endl;
	}

	~Base()
	{
		print();
	}
		
};

struct Derived : Base
{
	Derived()
	{
		print();
	}

	virtual void print()
	{
		std::cout << "Derived::print" << std::endl;
	}

	~Derived()
	{
		print();
	}
};

int main(int argc, char** argv)
{
	Derived d;
	Base& b = d;
	b.print();

	return EXIT_SUCCESS;
}
```

Вывод:

```C++
Base::print
Derived::print
Base::print
Derived::print
Base::print
```

Здесь стоит понимать, что в строке `b.print();` обращение идет к методу базового класса, потому что сам метод не виртуальный и он негде не переопределен.

_Вопрос:_

А что будет, если вывести следующий код?

```C++
#include <iostream>

struct Base
{
	Base()
	{
		print();
	}

	virtual void print()
	{
		std::cout << "Base::print" << std::endl;
	}

	~Base()
	{
		print();
	}
		
};

struct Derived : Base
{
	Derived()
	{
		print();
	}

	virtual void print()
	{
		std::cout << "Derived::print" << std::endl;
	}

	~Derived()
	{
		print();
	}
};

int main(int argc, char** argv)
{
	Derived d;
	Base& b = d;
	b.print();

	return EXIT_SUCCESS;
}
```

Вывод:

```C++
Base::print
Derived::print
Derived::print
Derived::print
Base::print
```

Теперь в строке `b.print();` идет обращение к переопределенному методу наследника. Переопределяется он неявно, но это происходит всегда, когда в базовом классе есть виртуальный метод, а в наследнике есть метод с идентичной сигнатурой.

Но почему в конструкторе и деструкторе не вызывается переопределенный метод?

Это все потому, что у любого метода, как бы отбрасывается слово `virtual` при вызове метода в конструкторе или деструкторе.

Слово `override` в методе может явно выкинуть ошибку компиляции, если вдруг базового метода не вышло найти в базовом классе.

_Вопрос:_

А что будет, если вывести следующий код?

```C++
#include <iostream>

struct Base
{
	Base()
	{
		print();
	}

	virtual void print()
	{
		std::cout << "Base::print" << std::endl;
	}

	~Base()
	{
		print();
	}
		
};

struct Derived : Base
{
	Derived()
	{
		print();
	}

	virtual void print()
	{
		std::cout << "Derived::print" << std::endl;
	}

	~Derived()
	{
		print();
	}
};

int main(int argc, char** argv)
{
	Base& b = new Derived;
	b->print();
	delete b;	

	return EXIT_SUCCESS;
}
```

Вывод:

```C++
Base::print
Derived::print
Derived::print
Derived::print
```

Все из-за того, что нет виртуального деструктора.

Так же стоит помнить, что если пометь метод как `final`, то переопределить его нельзя. А если же сам класс числится как `final`, то от этого класса нельзя будет наследоваться.

---

## Вопрос шестой.

_Вопрос:_

Что выведет данный код?

```C++
#include <iostream>

struct Base
{
	virtual void print(int value)
	{
		std::cout << "Base::print" << std::endl;
	}
};


struct Derived : Base
{
	virtual void print(long value)
	{
		std::cout << "Derived::print" << std::endl;
	}
};

int main(int argc, char** argv)
{
	Base* b = new Derived;
	b->print(100);
	return EXIT_SUCCESS; 
}
```

Вывод:

```C++
Base::print
```

Тут нет переопределения, только перегрузка. Сигнатуры разные. Что б ловить такие ошибки, и нужно использовать слово `override`.

---

## Вопрос седьмой.

_Вопрос:_

Что выведет данный код?

```C++
#include <iostream>

struct Base
{
    Base()
    {
        print();
    }

    void print()
    {
        print_impl();
    }

private:

    virtual void print_impl() = 0;
    
};

struct Derived : Base
{
    void print_impl() override
    {
        std::cout << "print_impl" << std::endl;
    }
};

int main (int argc, char** argv)
{
    Derived derived;
    return EXIT_SUCCESS;
}
```

Код выведет:

```Bash
pure virtual method called
terminate called without an active exception
Program terminated with signal: SIGSEGV
```

То есть, по сути, он просто упал. Почему же это произошло?

В данном коде, в конструкторе есть _NVI_ (_Non Virtual Interface_), смысл этой концепции в том, что за невиртуальной функцией происходит вызов виртуальной функции. Эдакое сокрытие вызова виртуальной функции. И предаставляя пользователю работу с невиртуальной функцией мы можем подменять логику, в классах наследниках.

Вспомним, что ключевое слово `virtual` просто отбрасывается в конструкторах/деструкторах, а значит идет попытка вызова функции без тела.

Поэтому код и падает.

_NVI_ - нужен тут для того, что бы запутать компилятор, потому что с ним не будет даваться ворнинг или ошибка про _pure virtual function_ в деструкторе.

---

## Вопрос восьмой.

_Вопрос:_

Что происходит в данном коде?

```C++
#include <iostream>

struct SuperBase
{
    int super_base_data;
};

struct Base1 : SuperBase
{
    int base_1_data;
};

struct Base2 : SuperBase
{
    int base_2_data;
};

struct Derived : Base1, Base2
{
    int derived_data;
};

int main (int argc, char** argv)
{
    Derived derived;
    derived.super_base_data = 100;
    return EXIT_SUCCESS;
}
```

В данном коде происходит ромбовидное наследование:

```C++
     SP
    / \
  B1   B2
    \ /
     D
```

И данный код упадет с ошибкой:

```C++
<source>:26:13: error: request for member 'super_base_data' is ambiguous
   26 |     derived.super_base_data = 100;
```

Есть несколько вариантов решения данной проблемы:

1. Явно указать класс, через который пойдет доступ к переменной базового класса:
    
```C++
derived.Base1::super_base_data = 100;
```
    
2. Определить класс `SuperBase` как виртуальное наследование.
    
```C++
struct Base1 : virtual SuperBase
{
	int base_1_data;
};

struct Base2 : virtual SuperBase
{
	int base_2_data;
};
```
    
    Как же это будет работать под капотом?
    
    Очень просто в инстансе `Base1` и `Base2` создастся по указателю `vb_ptr`. И этот указатель будет указывать на инстенс базового класса `SuperBase`.

---
## Вопрос девятый.

_Вопрос:_

Что выведет данный код?

```C++
#include <iostream>

struct Base
{
    void print(int value)
    {
        std::cout << "print: " << value << std::endl;
    }

    void print(const std::string& data)
    {
        std::cout << "print: " << data << std::endl;
    }
};

struct Derived : Base
{
    void print(int value)
    {
        std::cout << "print: " << value << std::endl;
    }
};

int main (int argc, char** argv)
{
    Derived derived;
    derived.print(100);
    return EXIT_SUCCESS;
}
```

Данный код выведет:

```C++
print: 100
```

Но какой именно `print(int)` будет вызван?

Такой вариант называется _Сокрытием Метода_ и вызываться будет `Derived::print(int)` , данный прием считается антипаттерном и приводит к тому, что до `Base::print(int)` и до `Base::print(std::string)` нельзя будет достучаться. То есть код `derived.print(”100”);` выдаст ошибку. Потому что если вдруг, хоть одно имя в наследнике совершает _Сокрытие_, то вся иерархия перегрузок сокрывается.

Для того, что бы часть открыть, а одно имя сокрыть, надо сделать так:

```C++
using Base::print;    
void print(int value)
{
	std::cout << "print: " << value << std::endl;
}
```

И тогда код `derived.print(”100”);` отработает нормально.

---
## Вопрос десятый.

_Вопрос:_

Что выведет данный код?

```C++
#include <iostream>

struct Data
{
    const int& get_value() const&
    {
        std::cout << "get_value_1" << std::endl;
        return m_value;
    }

    int get_value() const&&
    {
        std::cout << "get_value_2" << std::endl;
        return m_value;
    }

private:

    int m_value = 100;
    
};

void func(int value) {}

int main (int argc, char** argv)
{
    Data data;
    func(data.get_value());
    func(Data{}.get_value());
    return EXIT_SUCCESS;
}
```

Вывод будет следующий:

```C++
get_value_1
get_value_2
```

В данном коде есть перегрузка, но она идет не по константности, а “по амперсандности”

В первом случае вызывается `const int& get_value() const&` потому что объект считаеся _lvalue_.

Во втором случае вызывается `int get_value() const&&` потому что объект создается как _rvalue_. (У _rvalue_ нельзя взять адрес).

---
## Вопрос одиннадцатый.

_Вопрос:_

Что выведет данный код?

```C++
#include <iostream>

struct Data
{
    Data() = default;

    Data(const Data& other)
    {
        std::cout << "copy ctr" << std::endl;
    }

    Data(Data&& other)
    {
        std::cout << "move ctr" << std::endl;
    }
};

int main (int argc, char** argv)
{
    // 1
    {
        int value = 100;
        int moved_value = std::move(value);
        std::cout << "moved_value: " << value << std::endl;
    }
		// 2
    {
        Data data;
        Data moved_data = std::move(data);

        const Data const_data;
        Data const_moved_data = std::move(const_data);
    }
    return EXIT_SUCCESS;
}
```

Тут обычно любят спрашивать два вопроса:

1. Что тут будет выведено?
    ```C++
    moved_value: 100
    ```
    Это происходит потому что `std::move(value)` просто развернется  
    в  
    `static_cast<int&&>(value);` _**MOVE**_ **НИЧЕГО НЕ ПЕРЕМЕЩАЕТ, ПЕРЕМЕЩАТЬ ЧТО_ТО МОЖЕТ ТОЛЬКО ОПЕРАТОР КОПИРОВАНИЯ ИЛИ ОПЕРАТОР ПРИСВАИВАНИЯ**.
    
    А это не работает, потому что `std::move()` не работает с примитивными типами, хранящимися на стеке, или с `std::array<>`.
    
2. Что тут будет выведено?
    ```C++
    move ctr
    copy ctr
    ```
    std::move() в первом случае просто кастит `Data` к `Data&&` и именно поэтому перегрузка выбирает конструктор `Data(Data&& other)`.
    Во втором случае работает правило, не мувать константный объект, потому что это  бессмысленно, и просто будет передаваться сам этот объект без каста. И соответсвенно, будет вызываться конструктор копирования.
