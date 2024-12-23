
---

_Полное имени_ (Qualified name) - это такое имя, которое, находится по правую сторону от оператора разрешения области видимости.

Полное имя относится к:

1. Членам классов (включая статические, нестатические функции, так же типы, шаблоны и т.д.).
2. Членам пространства имен.
3. А так же счетчикам (enumerators)

Если с левой стороны от оператора :: ничего нет, то поиск (lookup), предполагает, что объявление имени происходит в глобальном пространстве имен.

```C++
#include <iostream>

int main()
{
	struct std {};

	std::cout << "Fails." // Error: применится поиск неполного имени для 'std'
						  // и найдется только структура в локальном скоупе
	::std::cout << "Ok."  // Ok: ::std найдет пространство имен std в глобальном
						  // скоупе, через \#include <iostream>
}
```

Перед тем как закончить поиск (lookup) сущности справа от ::, поиск (lookup) обязан быть завершен сначала для левой стороны от оператора ::.

Этот поиск может быть либо Qualified, либо Unqualified в зависимости от того, что стоит слева от оператора `::`, и работает только с типами: пространства имен, классами, перечислений или шаблонов. Во всех других случаях программа неправильно сформирована и упадет с ошибкой.

```C++
struct A
{
	static int n;
};

int main()
{
	int A;
	A::n = 42; // Ok: "Поиск неполного имени" класса А слева от оператора ::,
               // будет искать именно класс, игнорируя переменную int A;
	A b;       // Error: "Поиск неполного имени" класса A найдет переменную А
}

template<int>
struct B : A {};

namespace N
{

template<int>
void B();

int f()
{
	return B<0>::n; // Error: B<0>::n - это не тип
}

} // namespace N
```

---

### Частные случаи:

Когда “полное имя” это декларатор, тогда “поиск неполного имени” используется в идущем за “полным именем” декларатором, но этот поиск не используется в имени, которое идет до “полного имени”.

```C++
class X {};

constexpr int number = 100;

struct C
{
	class X {};
	static const int number = 50;
	static X arr[number];
};

X C::arr[number], brr[number]; // Error: Тут Х ищется "неполным поиском" и 
                               // находит ::Х, а не С::Х

C::X C::arr[number], brr[number]; // Ок: размер arr будет 50, к нему применяется
                                  // "полный поиск", а brr будет размера 100,
                                  // он ищется "неполным поиском"
```

  

Если за оператором **::** следует символ **~**, это значит что будет искаться деструктор (или псевдо-деструктор), в области, которая стоит по левую руку от оператора **::**.

```C++
struct C { typedef int I; };

typedef int I1, I2;

extern int *p, *q;

struct A { ~A(); };

typedef A AB;

int main()
{
    p->C::I::~I(); // Имя I после знака ~ будет искаться в той же области, что и
	               // I до :: (то же праивло действует и для C, поэтому найдется
				   // C::I)

	q->I1::~I2(); // Имя I2, будет искаться в области имени I1 (а так, как это
				  // та же область, что и I2, то и найдется ::I2)

	AB x;
	x.AB::~AB(); // Имя AB после ~ найдется в той же области, что и AB до ::
				 // (А так как это все происходит в глобальной области, 
				 // найдется ::AB)
}
```

---

## “Qualified look up” - с разными типами данных:

### Счетчик (enumerators):

Если “полный поиск” применяется к счетчикам, и имя счетчика расположено слева от оператора **::**, то справа обязано быть имя из области этого счетчика, иначе программа неправильно сформирована и упадет с ошибкой.

### Член класса (class members):

Если “полный поиск” применяется к имени класса/структуры/объединения, и имя класса стоит слева от оператора ::, то сущность, которая стоит справа от оператора :: будет искаться в области видимости класса, который стоит по левую руку от оператора ::.

Данное правило действует с некоторыми оговорками:

1. Деструктор всегда ищется так, как описано выше.
2. Оператор приведения типов в классе сначала будет искаться в области видимости класса, и если не найдет ничего - будет искаться в области, в которой произошел вызов.
3. Имена используемые в шаблонных аргументах ищутся в области вызова, а не в области самого шаблонного имени
4. Имена определенные как _using_ ищутся так же в области определения сначала, а не локально.

Если тип это класс или структура и по обе стороны от оператора :: будет стоять одно и то же имя - это значит, что вызовется конструктор этого типа.

```C++
struct A { A(); };

struct B : A { B(); };

A::A() {} // A::A - определяет конструктор, использованный в объявлении класса A
B::B() {} // B::B - определяет конструктор, использованный в объявлении класса B

B::A ba;  // B::A - определяет переменную типа A в классе B
A::A a;   // Error: Не определяет никакой тип, тк конфликтует с 
          // вызовом или определением конструктора

struct A::A a2; // Ok: поиск игнорирует так конструктор, и считает, что просто
                // объявляет член класса A в области класса A 
                // (that is, the injected-class-name)
```

“Полный поиск имени” можно так же использовать для доступа к скрытой вложенным объявлением методу класса. Вызов “полного имени” функции - никогда не будет виртуальным.

```C++
struct B { virtual void foo(); };

struct D : B { void foo() override; };

int main()
{
	D x;
    B& b = x;

	b.foo(); // Тут происходи вызов перзаписанного метода D::foo 
			 // (virtual dispatch)
	b.B::foo() // Тут происходи вызов виртуального метода B::foo
			   // (static dispatch) 
}
```

### Член пространства имен:

Если имя справа от оператора :: является именем пространства имен, либо если слева от оператора :: ничего нет (в этом случае просто имеется в виду глобальное пространство имен), тогда имя, стоящее справа от оператора :: будет искаться в области видимости этого пространства имен.

За исключением случая, когда имя используется как аргумент шаблона (а все агрументы шаблона ищутся там, где объявлен шаблон)

```C++
namespace N
{

template<typename T>
struct foo {};

struct X {};

}; // namespace N

N::foo<X> x; // Error: имя X ищется как ::X, а не как N::X
```

“Полный поиск имени” в пределах пространства имен _N_, для начала проходит по всем объявления внутри самого _N_ и всех inline-объявлений пространств. Если в этом наборе нет объявлений, то он рассматривает объявления во всех пространствах имен, названных с помощью-директив, найденных в _N_, и во всех транзитивных встроенных членах пространства имен _N_.

Правило применяется рекурсивно:

```C++
int x;

namespace Y
{

void f (float);
void h (int);

} // namespace Y

namespace Z
{

void h (double);

} // namespace Z

namespace A
{

using namespace Y;
void f (int);
void g (int);
int i;

} // namespace A

namespace B
{

using namespace Z;
void f (char);
int i;

} // namespace B

namespace AB
{

using namespace A;
using namespace B;
void g();

} // namespace AB

void h()
{
	AB::g(); // Сначала поиск произошел в namespace AB, там был найден 
		 // AB::g(void); И поэтому поиск в namespace A и namespace B
		 // не был произведен.

	AB::f(1); // Сначала поиск произошел в namespace AB, там ничего не нашлось
			  // Потом поиск произошел в A, потом в B
			  // Поиск нашел два варианта: A::f, B::f
			  // (Но не было поиска по Y, поэтому Y::f не был найден)
			  // Потом методом перегрузки был выбран A::f(int)

	AB::x++;  // Сначала поиск произошел в namespace AB, там ничего не нашлось
			  // Потом поиск произошел в A, потом в B, там ничего не нашлось
			  // Потом поиск произошел в Y, потом в Z, там тоже ничего не 
              // нашлось, поэтому выдается ошибка

	AB::i++   // Сначала поиск произошел в namespace AB, там ничего не нашлось
			  // Потом поиск произошел в A, потом в B, там нашлись A::i и B::i
              // так как имена идентичны - выдается ошибка.

	AB::h(16.8); // Сначала поиск произошел в namespace AB, там ничего не нашлось
                 // Потом поиск произошел в A, потом в B, там ничего не нашлось
                 // Потом поиск произошел в Y, потом в Z, там нашлось Y::h, Z::h
				 // Потом методом перегрузки был выбран Z::h(double) 
}
```

Так же позволяется для одного определения, находить его ромбовидным поиском:

```C++
namespace A 
{

int a; 

} // namespace A 
 
namespace B
{

using namespace A;

} // namespace B
 
namespace D
{ 

using A::a; 

} // namespace D
 
namespace BD
{

using namespace B;
using namespace D;

} // namespace BD
 
void g()
{
    BD::a++; // OK: Ошибки не будет, так как найдется один и тот же A::a, 
             // но через B и D. То есть найдется BD::B::A::a и BD::D::A::a
             // Имперически, это говорит о правиле, что если найдено два
             // одинаковых имени справа от ::, то если слева от :: будут
			 // разные имена - программа упадет.
}
```