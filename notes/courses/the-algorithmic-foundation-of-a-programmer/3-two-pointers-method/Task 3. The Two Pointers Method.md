
---

## Задача № 3

- Дано две строки
- Проверить, является ли вторая из них первой, в которой некоторые буквы, возможно, повторены
- Пример
	- `alex`, `aaleex` -> True
	- `saeed`, `saaed` -> False
	- `laiden`, `laiden` -> True

---

## Идея

- У задачи нет очевидного неэффективного решения
- Поставим по указателю в начало каждой строки
- Заведем инвариант - символы по указателю должны совпадать
- Будем выделять интервалы одинаковых символов и проверять, что во второй строке интервал оказался не короче, чем в первой

---
### Пример 1
Исходные данные:
![[algo-foundation-two-pointers-18.png]]
Мы выставляем два указателя в начало первой строки и в начало второй, потом выделяем диапазоны с буквами, которые равны - в данном случае это `a` и после этого проверяем, что второй диапазон оказался **не короче** первого (т.е. равен или больше):
![[algo-foundation-two-pointers-20.png]]
Проверили, все условия выполняются - переключаемся на символ `l`:
![[algo-foundation-two-pointers-21.png]]
Далее переходим к третьему символу `e`, проверяем его, так же не забываем проверять, что в конце второй строки букв не более, чем в первой:
![[algo-foundation-two-pointers-22.png]]
И потом переходим к последнему символу `x` и проверяем его как и все ранее:
![[algo-foundation-two-pointers-23.png]]

---
### Пример 2

Начнем с первого символа `s` и поставим укахаетли в начало слова:
![[algo-foundation-two-pointers-24.png]]
Далее для символов `a` тоже выполняются все условия:
![[algo-foundation-two-pointers-25.png]]
А вот для букв `e` уже ничего не выполянется, потому что мы выделили диапазон во второй строке и он оказался кароче, чем в первой - значит, инвариант нарушен:
![[algo-foundation-two-pointers-26.png]]

## Реализация

```cpp
std::size_t advance(const std::string& current, std::size_t position)
{
	do
	{
		++position;
	} while (position < current.size() 
		&& current[position] == current[position - 1]
	);

	return position;
}

bool isLongPressedName(string name, string typed)
{
	std::size_t second = 0;

	for (std::size_t first = 0; first < name.size();)
	{
		if (second >= typed.size()
			|| name[first] != typed[second]
		)
		{
			return false;
		}

		const auto next_first = advance(name, first);
		const auto next_second = advance(typed, second);
		if (next_first - first > next_second - second)
		{
			return false;
		}

		first = next_first;
		second = next_second;
	}    
	
	return second == typed.size();
}

```