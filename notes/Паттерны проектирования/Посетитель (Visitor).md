Шаблон проектирования Посетитель - используется для выполнения операции над группой объектов аналогичного типа или иерархией.

---

## Ресурсы:

1. [Статья про паттерн Посетитель](https://www.vishalchovatiya.com/double-dispatch-visitor-design-pattern-in-modern-cpp/).

---

## Краткий обзор содержания:

1. [[Посетитель (Visitor)]]
2. [[Посетитель (Visitor)]][[Вопросы задаваемые на собеседованиях- 2]][[Вопросы задаваемые на собеседованиях- 2]]
    1. [[Посетитель (Visitor)]]
    2. [[Посетитель (Visitor)]]
    3. [[Посетитель (Visitor)]]
3. [[Посетитель (Visitor)]]
4. [[Посетитель (Visitor)]]
5. [[Посетитель (Visitor)]]

---

## 1. Смысл паттерна.

_Определить новую операцию для группы похожих объектов, или для иерархии объектов._

- Классический Посетитель должен иметь некий компонент, который мы назовем `visitor` . И так же нужно добавить метод `visit()` в целую иерархию, для возможности прохождения по ней.
- После выполнения этих простых действий, нам больше не потребуется трогать иерархию. Поэтому эта иерархия может существовать сама по себе, и для добавления нового функционала нужна будет только новая сущность `visitor`. Данная возможность идеально соответствует принципам [SOLID](https://www.vishalchovatiya.com/single-responsibility-principle-in-cpp-solid-as-a-rock/), а конкретно - принципу “открытости/закрытости” и принципу “единственной ответственности”.

---

## 2. Пример реализации шаблона на `C++`.

Так как данный паттерн довольно сложен для первичного понимания, поступим следующим образом: сначала рассмотрим все возможные варианты этого паттерна, и только потом дойдем до классического варианта. Так будет проще понять важность Посетителя.

---

## 2.1. Назойливый Посетитель (Intrusive Visitor).

Предположим у нас есть иерархия документов:

```C++
class Document
{
public:
		virtual void add_to_list(const std::string &line) = 0;
};

class Markdown : public Document
{
public:
		void add_to_list(const std::string &line) { m_content.push_back(line); }

private:
		std::string              m_start = "* ";
		std::vector<std::string> m_content;
};

class HTML : public Document
{
public:
		void add_to_list(const std::string &line) { m_content.push_back(line); }

private:
		std::string              m_start = "<li>";
		std::string              m_end = "</li>";
		std::vector<std::string> m_content;
};
```

_И задача стоит следующая_: надо определить новые операции на существующей иерархией. К примеру мы хотим что бы все наследники `Document` могли выводиться на печать.

Задача поставлена и мы начинаем думать над её реализацией. Единственное, что приходит в голову на данном этапе - это добавить метод `print()` во всю иерархию класса. В голову то это пришло, но так не хочется этого делать: слишком много предстоит менять. Но дедлайны уже горят, поэтому применяем наивное решение:

```C++
class Document
{
public:
		virtual void add_to_list(const std::string &line) = 0;
        virtual void print() = 0;
};

class Markdown : public Document
{
public:
		void add_to_list(const std::string &line) { m_content.push_back(line); }
        void print()
        {
            for (auto &&item : m_content)
            {
                std::cout << m_start << item << std::endl;
            }
        }

private:
		std::string              m_start = "* ";
		std::vector<std::string> m_content;
};

class HTML : public Document
{
public:
		void add_to_list(const std::string &line) { m_content.push_back(line); }
        void print()
        {
            std::cout << "<ul>" << std::endl;
            for (auto &&item : m_content)
            {
                std::cout << "\t" << m_start << item << m_end << std::endl;
            }
            std::cout << "</ul>" << std::endl;           
        }
private:
		std::string              m_start = "<li>";
		std::string              m_end = "</li>";
		std::vector<std::string> m_content;
};
```

В принципе выглядит не так уж и плохо (хотя это и прямое нарушение принципа единственной ответственности), но это только для 2-3 классов в иерархии. Чем больше классов - тем больше неудобных правок. А если потом добавятся еще действия, типа `save()` , `process()`, `handle()` и так далее в наши 20 потенциальных классов в иерархии - это же можно свихнуться.

---

## 2.2. Отражающий Посетитель (Reflective Visitor).

Поэтому подумав еще немного, мы придумываем следующую реализацию:

```C++
class Document
{
public:
		virtual void add_to_list(const std::string &line) = 0;
};

class Markdown : public Document
{
public:
		void add_to_list(const std::string &line) { m_content.push_back(line); }

        friend class DocumentPrinter;

private:
		std::string              m_start = "* ";
		std::vector<std::string> m_content;
};

class HTML : public Document
{
public:
		void add_to_list(const std::string &line) { m_content.push_back(line); }

        friend class DocumentPrinter;

private:
		std::string              m_start = "<li>";
		std::string              m_end = "</li>";
		std::vector<std::string> m_content;
};

class DocumentPrinter
{
public:
    static void print(Document* e)
    {
        if (auto md = dynamic_cast<Markdown*>(e))
        {
            for (auto &&item : md->m_content)
            {
                std::cout << md->m_start << item << std::endl;
            }
        }
        else if (auto hd = dynamic_cast<HTML*>(e))
        {   
            std::cout << "<ul>" << std::endl;
            for (auto &&item : hd->m_content)
            {
                std::cout << "\t" << hd->m_start << item << hd->m_end << std::endl;
            }
            std::cout << "</ul>" << std::endl;
        }
    }

};

int main()
{
    Document *d = new HTML;
    d->add_to_list("This is line");
    DocumentPrinter::print(d);
    return EXIT_SUCCESS;
}
```

Как упоминалось ранее, для соблюдения принципа “Единственной Ответственности” мы создали отдельный класс для функционала печати. Тут все ок.

Но вот эти `dynamic_cast<>()` , так еще и в статическом методе, будут тормозить программу. Особенно, если вспомнить, что потенциально в классе будет 20 наследников - выйдет что ветвление `if-else` будет просто огромным.

---

## 2.3. Классический Посетитель (Visitor).

Потыкав палкой во все вышеописанные полумеры, отказавшись от хардкодинга всей иерархии методов, от `if-else` и `dynamic_cast<>()` мы пришли к следующему решению:

```C++
/* ------------------------ Added Visitor Classes -------------------------- */
class DocumentVisitor
{
public:
    virtual void visit(class Markdown *) = 0;
    virtual void visit(class HTML *) = 0;
};

class DocumentPrinter : public DocumentVisitor
{
public:
    virtual void visit(class Markdown *);
    virtual void visit(class HTML *);
};

/* ------------------------------------------------------------------------- */

class Document
{
public:
    virtual void add_to_list(const std::string &line) = 0;
    virtual void visit(DocumentVisitor*) = 0; // <<<<<<<<<<--------------------
};

class Markdown : public Document
{
public:
    void add_to_list(const std::string &line) { m_content.push_back(line); }
    void visit(DocumentVisitor* dv) { dv->visit(this); } // <<<<<<<<<<--------
    friend DocumentPrinter;

private:
    std::string              m_start = "* ";
    std::vector<std::string> m_content;
};

class HTML : public Document
{
public:
    void add_to_list(const std::string &line) { m_content.push_back(line); }
    void visit(DocumentVisitor* dv) { dv->visit(this); } // <<<<<<<<<<---------
    friend DocumentPrinter;

private:
    std::string              m_start = "<li>";
    std::string              m_end = "</li>";
    std::vector<std::string> m_content;
};

/* ----------------------- Added Visitor Methods ---------------------------- */
void DocumentPrinter::visit(Markdown* md)
{
    for (auto &&item : md->m_content)
    {
        std::cout << md->m_start << item << std::endl;
    }
}

void DocumentPrinter::visit(HTML* hd)
{
    std::cout << "<ul>" << std::endl;
    for (auto &&item : hd->m_content)
    {
        std::cout << "\t" << hd->m_start << item << hd->m_end << std::endl;
    }
    std::cout << "</ul>" << std::endl;
}
/* ------------------------------------------------------------------------ */

int main()
{
    Document *d = new HTML;
    d->add_to_list("This is line");
    d->visit(new DocumentPrinter);
    return EXIT_SUCCESS;
}
```

Мы добавили двухслойное косвенное обращение к нужному нам функционалу. И таким образом мы перестали нарушать принцип “Единственной ответственности”, и принцип “Открытия/Закрытия”. Данный хак называется: [[notes/Вопросы задаваемые на собеседованиях_external/C++. Что такое двойная диспетчеризация|C++. Что такое двойная диспетчеризация]].

---

## 3. Паттерн Посетитель в современном C++.

```C++
class Document
{
public:
    virtual void add_to_list(const std::string &line) = 0;
};

class Markdown : public Document
{
public:
    void add_to_list(const std::string &line) { m_content.push_back(line); }
    friend class DocumentPrinter;

private:
    std::string              m_start = "* ";
    std::vector<std::string> m_content;
};

class HTML : public Document
{
public:
    void add_to_list(const std::string &line) { m_content.push_back(line); }
    friend class DocumentPrinter;

private:
    std::string              m_start = "<li>";
    std::string              m_end = "</li>";
    std::vector<std::string> m_content;
};


/* --------------------------------- Visitor ---------------------------------- */
class DocumentPrinter
{
public:
    void operator() (Markdown &md)
    {
        for (auto &&item : md.m_content)
        {
            std::cout << md.m_start << item << std::endl;
        }
    }
    void operator()(HTML &hd){
        std::cout << "<ul>" << std::endl;
        for (auto &&item : hd.m_content)
        {
            std::cout << "\t" << hd.m_start << item << hd.m_end << std::endl;
        }
        std::cout << "</ul>" << std::endl;
    }
};
/* ---------------------------------------------------------------------------- */

using document = std::variant<Markdown, HTML>;

int main()
{
    HTML hd;
    hd.add_to_list("This is line");
    document d = hd;
    DocumentPrinter dp;
    std::visit(dp, d);
    return EXIT_SUCCESS;
}
```

```Bash
<ul>
	<li>This is line</li>
</ul>
```

Строка `using document = std::variant<Markdown, HTML>;` подразумевает, что можно в тип `document` может быть либо типом `Markdown`, либо типом `HTML` (но только одним за раз).

Так же современный `C++` (начиная с 17-го стандарта) имеет в стандартной библиотеке функцию `std::visit()` , которая принимает вызываемые объекты, как объект типа `DocumentPrinter` .

---

## 4. Достоинства паттерна Посетитель.

1. Соблюдение принципа “Единственной ответственности”, а значит разделения специфичной для типа логики в отдельные сущности/классы.
    
    В нашем случае это добавление класса `DocumentPrinter`, который отвечает только за действия над объектами.
    
2. Соблюдения принципа “Открытости/Закрытости”, это значит, что новая функциональность может быть добавлена без изменения заголовков классов.
    
    Как в примере выше, мы изначально добавили метод `visit()` во всю иерархию объектов - а потом вынесли метод, как отдельную сущность в логику класса `DocumentPrinter` . А значит, что если нам понадобится не только принтер, но сканер, мы просто создадим класс по похожей логике: `DocumentScanner`.
    
3. Так как функциональность разбивается гораздо проще проводить юнит-тестирование.
4. Вырастает производительность, в отличии от методов с `dynamic_cast`, `typeid()` или `enum`/`string` сравнение.

---

## 5. Заключение.

**Когда я должен использовать шаблон Посетитель?**

Шаблон проектирования Посетитель весьма полезен, когда наши требования постоянно меняются, что также влияет на несколько классов в иерархии наследования.

**Каков типичный вариант использования шаблона дизайна посетителя?**

- Для замене `dynamic_cast<>`, `typeid()` и т.д.
- Для обработки коллекции различных типов объектов.
- Фильтрация различных типов объектов из коллекций.

**Разница между шаблоном Посетитель и Декоратор?**

_Декоратор_ (шаблон структурного проектирования) работает над объектом, расширяя существующую функциональность.  
  
_Посетитель_ (шаблон поведенческого проектирования) работает с иерархией классов, где вы хотите запустить другой метод, основанный на конкретном типе, но избегающий операторов `dynamic_cast<>()` или `typeof()`.