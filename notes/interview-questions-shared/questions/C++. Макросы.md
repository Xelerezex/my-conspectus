
---
## Вопрос первый.

_Вопрос:_

Что выведет данный код?

```C++
#include <iostream>

#define MUL(x) (x * 5)

int main(int argc, char** argv)
{
	const int x = 5;
	const int result = MUL(x + 5);

	std::cout << "Result: " << result << std::endl;

	return EXIT_SUCCESS;
}
```

```C++
Result: 30
```

Так почему же получился такой вывод?

Просто компилятор после препроцессинга строку `const int result = MUL(x + 5);` преобразует в строку `const int result = (x + 5 * 5);`

Исходя из вышеописанного, хорошей практикой считается обрамлять все элемента макроса скобками

```C++
#define MUL(x) ((x) * (5))
```

---
## Вопрос второй.

_Вопрос:_

Что выведет данный код?

```C++
#include <iostream>

#define ABS(x) ((x) < 0 ? -(x) : (x))

int main(int argc, char** argv)
{
	int x = 5;
	const int result = ABS(x++);

	std::cout << "Result: " << result << std::endl;
	std::cout << "X:      " << x << std::endl;

	return EXIT_SUCCESS;
}
```

```C++
Result: 6
X:      7
```

Просто компилятор после препроцессинга строку `const int result = ABS(x++);` преобразует в строку `const int result = ((x++) < 0 ? -(x++) : (x++));`

Как мы видим, оператор `++` вызывается дважды.

---
## Вопрос третий.

_Вопрос:_

Какая потенциальная проблема есть в данном коде?

```C++
#include <iostream>

#define CHECK_EVEN_NUMBER(number)    \
	if ((number) % 2 == 0)           \
		std::cout << "number is even" << std::endl;

int main(int argc, char** argv)
{
		int number = 1;

		if (number != 0)
			CHECK_EVEN_NUMBER(number);
		else
			std::cout << "number is zero" << std::endl;
	
		return EXIT_SUCCESS;
}
```

После препроцессинга макрос раскроется в следующий код:

```C++
if (number != 0)
	if ((number) % 2 == 0)
		std::cout << "number is even" << std::endl;
	else
		std::cout << "number is zero" << std::endl;
```

И избежать этого можно только через подход `do {} while(true)` внутри макроса:

```C++
#define CHECK_EVEN_NUMBER(number)                       \
	do {                                                \
		if ((number) % 2 == 0)                          \
			std::cout << "number is even" << std::endl; \
	} while(false)                                      \
```

