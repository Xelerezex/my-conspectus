---

### Ресурсы:

1. [Что такое двойная диспетчеризация?](https://www.vishalchovatiya.com/double-dispatch-in-cpp/)
2. [Что такое двойная диспетчеризация?](https://dzone.com/articles/double-dispatch-in-c)
3. [Принцип единственной ответственности.](https://www.vishalchovatiya.com/single-responsibility-principle-in-cpp-solid-as-a-rock/)

---

## Краткий обзор содержания:

1. [[C++. Что такое двойная диспетчеризация]]
2. [[C++. Что такое двойная диспетчеризация]]
3. [[C++. Что такое двойная диспетчеризация]]
4. [[C++. Что такое двойная диспетчеризация]]
5. [[C++. Что такое двойная диспетчеризация]]
    1. [[C++. Что такое двойная диспетчеризация]]
6. [[C++. Что такое двойная диспетчеризация]]`[[C++. Что такое двойная диспетчеризация]]`[[C++. Что такое двойная диспетчеризация]]
7. [[C++. Что такое двойная диспетчеризация]]
8. [[C++. Что такое двойная диспетчеризация]]`[[C++. Что такое двойная диспетчеризация]]`[[C++. Что такое двойная диспетчеризация]]`[[C++. Что такое двойная диспетчеризация]]`[[C++. Что такое двойная диспетчеризация]]`[[C++. Что такое двойная диспетчеризация]]`[[C++. Что такое двойная диспетчеризация]]
9. [[C++. Что такое двойная диспетчеризация]]
10. [[C++. Что такое двойная диспетчеризация]]
11. [[C++. Что такое двойная диспетчеризация]]

---

_Двойная диспетчеризация_ - это механизм в `C++` , который отправляет вызов функции в другую конкретную функцию, в зависимости от их типов, во время выполнения программы.

Другими словами, это вызов функции по средствам двух разных виртуальных таблиц соответствующих разных объектов.

---

## 1. Мотивация.

Как только мы начали проектировать какой-то класс, и у нас встала задача положить его в контейнер - указатель на базовый класс выглядит вполне вменяемым решением:

```C++
class Animal
{
public:
    virtual const char *name() = 0;
};

using AnimalList = std::vector<Animal*>;
```

Потом нам захотелось добавить добавить несколько наследников от класса `Animal` . И нам надо только возвращать имена этих животных, не зная их типа (нужен только тип `Animal`):

```C++
class Cat : Animal
{
public:
    const char *name() { return "Cat"; }
};

class Dog : Animal
{
public:
    const char *name() { return "Dog"; }
};
```

Но вот к нам приходит клиент и говорит, что хочет видеть в программе следующую логику:

Существует класс `Person` , и если он встречает на своем пути `Dog` - он должен испугаться и убежать. А если он видит `Cat` - должен погладить её. И вот перед нами встает проблема - потому что с условиями выше, нам надо знать тип класса-наследника…

---

## 2. Идентификация типа во время выполнения.

Поэтому мы подумали, раз нам нужно идентифицировать тип во время выполнения и типов-наследников у нас немного, реализуем это так:

```C++
class Person 
{
public:
    void ReactTo(Animal *_animal) 
    {
        if (dynamic_cast<Dog *>(_animal))
            RunAwayFrom(_animal);
        else if (dynamic_cast<Cat *>(_animal))
            TryToPet(_animal);
    }

    void RunAwayFrom(Animal *_animal) 
		{ 
				cout << "Run Away From :" << _animal->name() << endl; 
		}
    
		void TryToPet(Animal *_animal)    
		{ 
				cout << "Try To Pet    :" << _animal->name() << endl; 
		}
};
```

Но потом клиент хочет добавить еще и `Horse` , получается как-то так:

```C++
//...
void ReactTo(Animal *_animal) 
{
		if (dynamic_cast<Dog *>(_animal))
        RunAwayFrom(_animal);
    else if (dynamic_cast<Cat *>(_animal))
        TryToPet(_animal);
		else if (dynamic_cast<Horse *>(_animal))
        TryToRide(_animal);
}
//...
```

А потом просят добавить еще немного животных, и получается такая ситуация:

```C++
void Person::ReactTo(Animal *_animal) 
{
    if (dynamic_cast<Dog *>(_animal) || dynamic_cast<Gerbil *>(_animal)) {
        if (dynamic_cast<Dog *>(_animal) && dynamic_cast<Dog>()->GetBreed() == DogBreed.Daschund) // Daschund's are the exception
            TryToPet(_animal);
        else
            RunAwayFrom(_animal);
    }
    else if (dynamic_cast<Cat *>(_animal) || dynamic_cast<Pig *>(_animal))
        TryToPet(_animal);
    else if (dynamic_cast<Horse *>(_animal))
        TryToRide(_animal);
    else if (dynamic_cast<Lizard *>(_animal))
        TryToFeed(_animal);
    else if (dynamic_cast<Mole *>(_animal))
        Attack(_animal)
    // etc.
}
```

И такое уродство будет долго продолжаться, все эти `dynamic_cast<>()` выглядят неправильно. Потом, конечно, можно попробовать вариант с `typeid()` , но в коде будет такая же грязь.

---

## 3. Использование полиморфизма.

В какой-то момент мы вспомнили, что можно присвоить типы объектам:

```C++
enum class AnimalType { Dog, Cat };

struct Animal
{
    virtual const char *name() = 0;
    virtual AnimalType type() = 0;
};

struct Cat : Animal
{
    const char *name() { return "Cat"; }
    AnimalType type() { return AnimalType::Cat; }
};

struct Dog : Animal
{
    const char *name() { return "Dog"; }
    AnimalType type() { return AnimalType::Dog; }
};

struct Person
{
    void ReactTo(Animal *_animal)
		{
        if (_animal->type() == AnimalType::Cat)
            TryToPet(_animal);
        else if (_animal->type() == AnimalType::Dog)
            RunAwayFrom(_animal);
    }

    void RunAwayFrom(Animal *_animal) { cout << "Run Away From " << _animal->name() << endl; }
    void TryToPet(Animal *_animal) { cout << "Try To Pet " << _animal->name() << endl; }
};

int main() {
    Person p;

    Animal *animal_0 = new Dog;
    p.ReactTo(animal_0);

    Animal *animal_1 = new Cat;
    p.ReactTo(animal_1);
    return 0;
}
```

Так мы получим прирост в производительности, но все еще будет проблема с огромным хвостом из `if/else` .

---

## 4. Более функциональный и модульный подход.

```C++
using PersonMethodPtr = void (Person::*)(Animal *);
using ReactionHash = unordered_map<AnimalType, PersonMethodPtr>;

void Person::ReactTo(Animal *_animal)
{
    static const ReactionHash reactionFunctions{
        {AnimalType::Cat, &TryToPet},
        {AnimalType::Dog, &RunAwayFrom},
        // etc.
    };

    reactionFunctions[_animal->type()](_animal);
}
```

Метод неплохой, но тут мы неявно переписываем [[С++. Что такое виртуальная таблица методов]], которую итак создает компилятор.

---

## 5. Единовременная диспетчеризация.

Поэтому, вместо того, что бы хранить какое-то поле с типом класса или использовать RTTI (`typeid()`). Мы можем использовать посредника, который будет направлять вызовы функций к надлежащему поведению.

```C++
using AnimalList = std::vector<class Animal*>;

class Animal
{
public:
    virtual string name() = 0;
    virtual void Visit(class ReactVisitor *visitor) = 0;
};

class ReactVisitor
{
public:
    class Person *person = nullptr;
};

class Person
{
public:
    void ReactTo (Animal *_animal)
    {
        ReactVisitor visitor{this};
        _animal->Visit(&visitor);
    }

    void RunAwayFrom(Animal *_animal) 
		{ cout << "Run away from " << _animal->name() << endl; }

    void TryToPet(Animal *_animal) 
		{ cout << "Try to pet " << _animal->name() << endl; }
};

class Cat : public Animal
{
public:
    string name() { return "Cat"; }
    void Visit(ReactVisitor *visitor) { visitor->person->TryToPet(this); }
};

class Dog : public Animal
{
public:
    string name() { return "Dog"; }
    void Visit(ReactVisitor *visitor) { visitor->person->RunAwayFrom(this); }
};

int main() 
{
    Person p;
    AnimalList animals = {new Dog, new Cat};
    
    for(auto&& animal : animals)
        p.ReactTo(animal);    
    
    return 0;
}
```

Что бы была возможность посреднику направлять вызовы функций, мы должны добавить метод `Visit(ReactVisitor*)` в классы наследники, который принимает посредника в виде аргумента. А потом мы добавляем правильные реакции для каждого типа и для `Dog`, и для `Cat`.

---

## 5.1. Проблемы, связанные с подходом единой диспетчеризации.

Из реализации выше возникает два вопроса:

1. Почему класс `Dog` должен диктовать классу `Person` как реагировать на него? Мы допустили обращение к внутренним данным класса `Person` , а значит, нарушили инкапсуляцию.
2. И что если у класса `Person` есть другие модели поведения, которые нужно произвести? Получается мы будем добавлять по новому виртуальному методу в базовый класс для каждого из них?

---

## 6. Двойная диспетчеризация в `C++`.

Обойти вышеперечисленные проблемы одиночной диспетчеризации нам поможет добавления еще одного слоя косвенных ссылок.

```C++
using AnimalList = std::vector<class Animal*>;

/* -------------------------------- Added Visitor Classes ------------------------------- */
class AnimalVisitor 
{
public:
    virtual void Visit(class Cat*) = 0;
    virtual void Visit(class Dog*) = 0;
};

class ReactVisitor : public AnimalVisitor
{
public:
    ReactVisitor(class Person* p) : person{p} {}
    void Visit(class Cat *c);
    void Visit(class Dog *d);

    class Person* person = nullptr;
};
/* --------------------------------------------------------------------------------------- */

class Animal
{
public:
    virtual string name() = 0;
    virtual void Visit(class AnimalVisitor* visitor) = 0;
};

class Cat : public Animal
{
public:
    string name() { return "Cat"; }
    void Visit(AnimalVisitor* visitor) { visitor->Visit(this); } // 2nd dispatch <<---------
};

class Dog : public Animal
{
public:
    string name() { return "Dog"; }
    void Visit(AnimalVisitor* visitor) { visitor->Visit(this); } // 2nd dispatch <<---------
};

class Person
{
public:
    void ReactTo(Animal *_animal)
    {
        ReactVisitor visitor{this};
        _animal->Visit(&visitor); // 1st dispatch <<---------
    }

    void RunAwayFrom(Animal *_animal) { cout << "Run Away From " << _animal->name() << endl; }
    void TryToPet(Animal *_animal)    { cout << "Try To Pet " << _animal->name() << endl; }
};

/* -------------------------------- Added Visitor Methods ------------------------------- */
void ReactVisitor::Visit(Cat *cat)  // Finally comes here <<-------------
{
    person->TryToPet(cat);
}
void ReactVisitor::Visit(Dog *dog)   // Finally comes here <<-------------
{ 
    person->RunAwayFrom(dog);
}
/* --------------------------------------------------------------------------------------- */

int main() 
{
    Person p;

    for(auto&& animal : AnimalList{new Dog, new Cat})
        p.ReactTo(animal);
    
    return 0; 
}
```

```Bash
Run Away From Dog
Try To Pet Cat
```

Выше показано, что вместо того, что бы обращаться напрямую к `ReactVisitor` мы используем `AnimalVisitor` как еще один слой для ссылок. И метод `visit(AnimalVisitor *)`в классах `Cat` и `Dog` принимают `AnimalVisitor` как параметр.

Такой подход дает нам два преимущества:

1. Мы не должны описывать поведения класса `Person` в классах `Cat` и `Dog` , таким образом не нарушается инкапсуляция.
2. Мы выделяем реакцию класса `Person` в отдельный класс `ReactVisitor` , а значит, мы следуем [принципу единственной ответственности.](https://www.vishalchovatiya.com/single-responsibility-principle-in-cpp-solid-as-a-rock/)

---

## 7. Как работает механизм двойной диспетчеризации?

Анимация того, как будет выглядеть стек-фрейм при двойной диспетчеризации.

![[double-dispatch-cpp-stack-frame-vishal-chovatiya.gif]]

  

  

---

## 8. Альтернативный подход к двойной диспетчеризации в современном `C++` с использованием `std::variant` и `std::visit` .

```C++
struct Animal
{
		virtual string name() = 0;
};

struct Cat : Animal
{
		string name() { return "Cat"; }
};

struct Dog : Animal
{
		string name() { return "Dog"; }
};

struct Person
{
		void RunAwayFrom(Animal* animal) { cout << "Run away from " << animal->name() <<endl; }
		void TryToPet(Animal* animal) { cout << "Try to pet " << animal->name() << endl; }
};

struct ReactVisitor
{
		void operator() (Cat *c) { person->TryToPet(c); }
		void operator() (Dog *d) { person->RunAwayFrom(d); }
		Person* person = nullptr;
};

using animal_ptr = std::variant<Cat*, Dog*>;

int main()
{
		Person p;
		ReactVisitor rv{&p};

		for (auto&& animal : vector<animal_ptr>({new Dog, new Cat}))
				std::visit(rv, animal);
}
```

```Bash
Run away from Dog
Try to pet Cat
```

1. `std::variant` выполняет схожую логику как `union` - можно достучаться либо до класса `Cat`, либо до класса `Dog` , в зависимости от того, кто спрашивает.
2. А так же в более новых стандартах `C++` (начиная с 17-го) появился класс `std::visit`, который получает на вход вызываемый объект класса `ReactVisitor` с функциями вызова, перегруженными по количеству типов в `std::variant` . И именно `std::visit` реализует в примере выше двойную диспетчеризацию.

---

## 9. Преимуществ механизма двойной диспетчеризации.

1. Соблюдение принципа единой ответственности - означает разделение специфичной для типа логики в отдельной сущности/классе. В нашем случае `ReactVisitor` обрабатывает реакцию для разных типов животных.
2. Соблюдение принципа _“Open-Closed Principle”_ - значит, что новая функциональность может быть добавлена без изменения заголовков классов в момент, когда мы вставили метод `visit()` . При этом, если хочется вставить новый метод `sound()` - надо будет добавить отдельный класс `SoundVisitor`.
3. Полезно при юнит-тестировании приложения.
4. Вырастает производительность в сравнении с использованием `dynamic_cast`, `typeid()` или же сравнений (==) `enum` или `string`.

---

## 10. Пример использования механизма двойной диспетчеризации.

1. Сортировка смешанного набора объектов: Вы можете реализовать фильтрацию с двойной диспетчеризацией. Например, “Дайте мне всех кошек из вектора`<Animal*>`“.
2. Можно добавить дополнительные функциональные возможности ко всей иерархии наследования, не изменяя ее снова и снова, например. если вы хотите добавить метод `sound()` для каждого отдельного животного, вы можете создать `SoundVisitor`, а остальная часть редактирования выполняется как тот же `ReactVisitor`.
3. Системы обработки событий, которые используют как тип события, так и тип объекта-приемника для вызова правильной процедуры обработки событий.
4. Адаптивные алгоритмы столкновения обычно требуют, чтобы столкновения между разными объектами обрабатывались по-разному. Типичный пример - игровая среда, где столкновение между космическим кораблем и астероидом вычисляется иначе, чем столкновение между космическим кораблем и космической станцией.

---

## 11. Заключение

У каждого решения есть свои преимущества и недостатки, и выбор одного из них зависит от конкретных потребностей вашего проекта. `C++` создает уникальные задачи при разработке таких высокоуровневых абстракций, поскольку он сравнительно жесткий и статически типизированный. Абстракции в `C++` также, как правило, стремятся быть как можно более дешевыми с точки зрения производительности во время выполнения и потребления памяти, что добавляет еще дополнительного усложнения.