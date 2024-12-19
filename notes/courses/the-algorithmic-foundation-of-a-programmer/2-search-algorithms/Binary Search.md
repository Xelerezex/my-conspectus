
---
Метод двоичного поиска применяется когда нет никакой информации о ключах и массив чисел _отсортирован_. Инвариант успеха данного алгоритма требует, что бы входное множество было упорядоченным (отсортированным).
Быстродествие данного алгоритма зависит от значения, которое мы выбираем _таргетным_, если мы к примеру всегда будем брать крайние (левые или правые) элементы или рандомные - то алгоритм в худщем отработает за $O(N)$.
Если же каждый раз мы будем брать именно центральный элемент, то каждый раз количество обрабатываемых элементов будет уменьшаться вдвое, а значит в худшем случае отработает все за $O(logN)$.

---
## Описание алгоритма
1. Сравниваем искомый элемент с центральным.
2. Если они равны, поиск завершается успехом.
3. Если искомый элемент меньше центрального, отбрасываем правую часть массива.
4. Иначе - левую.
5. Если массив не пуст, переходим к шагу 1, иначе поиск завершается неудачей.

![[binary-and-linear-search-animations.gif]]

**Дисклеймер:** Оценка работы данного алгоритма не просто $O(logN)$, а $O(logN)$ **сравнений**.
Потому что каждое **сравнение** тоже имеет свою сложность. Просто все обычно как на гифке выше представляют себе поиск числа `int`, но а если у нас будет число представлено буквами? К примеру `twelve` и при сравнении сначала надо прочитать строку, это уже займет $O(M)$, и потом сопоставить ее с числом. В таком кейсе итоговая оценка будет $O(M\ logN)$.
### Лучший случай
![[linear-vs-binary-search-best-case.gif]]
### Худший случай
![[linear-vs-binary-search-worst-case.gif]]

---
## Базовая реализация алгоритма

```cpp
#include <iostream>
#include <vector>

int BinarySearch(
    const std::vector<int>& data,
    std::size_t target
)
{
    std::size_t left = 0;
    std::size_t right = data.size() - 1;

    while (left <= right)
    {
        std::size_t middle = left + ((right - left) / 2);
        if (target == data[middle])
        {
            return middle;
        }
        if (target < data[middle])
        {
            right = middle - 1;
        }
        else
        {
            left = middle + 1;
        }
    }
    return -1;
}

int main()
{
    const std::vector<int> values {
        1, 5, 8, 12, 16, 22, 34, 45, 53, 62
    };
    const std::vector<int> test {
        0, 1, 2, 5, 6, 8, 9, 12, 13, 16, 17, 22,
        23, 34, 35, 45, 46, 53, 54, 62, 63
    };
    for (const auto& value : test)
    {
        std::cout << value << ": " << BinarySearch(values, value) << '\n';
    }
}
```
Данная реализация двоичного поиска гарантировано отыщет нужный нам элемент за $O(log N)$, но если входной массив все еще будет упорядочен, но при этом там появятся дубли?
Например `const std::vector<int> values { 1, 5, 8, 8, 8, 8, 8, 8, 8, 12, 16, 22 };
В таком случае у нас есть 2 устойчивых варианта, что мы можем вывести - это начало или конец подмножества дублей. То есть либо 8 с индексом 2, либо 8 с индексом 8.
Сначала рассмотрим вариант нахождения нижней границы
### Lower Bound (Бинарный поиск нижней границы на данных с повторением)


```cpp
#include <iostream>
#include <vector>

int LowerBound(
    const std::vector<int>& data,
    std::size_t target
)
{
    std::size_t left = 0;
    std::size_t right = data.size();
  
    while (left < right)
    {
        std::size_t middle = left + ((right - left) / 2);
        if (target <= data[middle])
        {
            right = middle;
        }
        else
        {
            left = middle + 1;
        }
    }
    if (right < data.size() && target == data[right])
    {
        return right;
    }
    return -1;
}


int main()
{
    const std::vector<int> values {
        1, 2, 3, 5, 5, 5, 5, 5, 5,
        5, 5, 7, 7, 7, 7, 9, 9, 11
    };
    const std::vector<int> test {
        0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12
    };
    for (const auto& value : test)
    {
        std::cout << value << ": " << LowerBound(values, value) << '\n';
    }
}
```

Изменения относительно эталонного бинарного поиска:
1. Первым делом в отличии от прошлого алгоритмы - мы удалили проверку `if (target == data[middle])`, потому что теперь просто найти значение нам не достаточно.
2.  Если у нас возникла ситуация, что `target == data[middle]` в какой части массива `data` нам надо продолжать поиск _самого левого значения_ `target`? Ну конечно же в левой часть. То есть мы отбрасываем правую часть меняя `right = middle;`. А значит теперь условие `if (target < data[middle])` будет выглядеть как `if (target <= data[middle])`.
3. Теперь `return` с выводом результирующего индекса будет за пределами основного цикла.  Запишем это так:
```cpp
    if (right < data.size() 
	    && target == data[right]
	)
	{
		return right + 1;
	}
```
---
## Алгоритмы бинарного поиска из стандартной библиотеки

В стандартной библиотеке языке так же присутсвуют все эти реализации базовых алгоритмов, и конечно на практике лучше использовать именно их, так как с высокой верятностью будет меньше ошибок допущено
### `std::binary_search()`

```cpp
template<
	class ForwardIt, 
	class T = typename std::iterator_traits,
	lass Compare
>
bool binary_search(
	ForwardIt first,   // Начало массива
	ForwardIt last,    // Конец массива
	const T& value,    // Значение, которое мы ищем
	Compare comparator // Функция компаратор
)
{
	first = std::lower_bound(
		first,
		last,
		value,
		comparator
	);
	return first != last // Первый и последни итератор не должны быть равны
		&& !comparator(value, *first); // И таргетное значение не должно быть меньше чем полученное значению начального итератора
}
```

Стоит обратить внимание, что реализации `std::binary_search()` - это буквально обвязка над алгоритмом `lower_bound()`, с той лишь разницой что алгоритм возвращает `bool` - присутсвует ли таргетное значение в массиве или нет.
И значение присутсвует в массиве при условии, что первый итератор не равен второму и таргетное значение не равно первому итератору. То есть `first != last; value >= *first`. Этого достаточно, что убедить в том, что в данном массиве есть такое значение.
### `std::lower_bound()`

```cpp
template<
	class ForwardIt, 
	class T = typename std::iterator_traits<ForwardIt>::value_type,
	class Compare
>
ForwardIt lower_bound(
	ForwardIt first, 
	ForwardIt last, 
	const T& value, 
	Compare comparator
)
{
	ForwardIt currentIterator; // Аналог задаваемого нами ранее элемента middle
	typename std::iterator_traits<ForwardIt>::difference_type count; // Это по факту просто число 
	typename std::iterator_traits<ForwardIt>::difference_type step; // И это тоже
	count = std::distance(first, last); // Первоначальное количество элементов, по которым будет идти поиск

	while (count > 0)
	{
		 currentIterator = first;
		 step = count / 2;
		 std::advance(currentIterator, step); // Передвигаем наш итератор на шаг (который равен половине от нынешнего количества элементов, по которым производится поиск)
		if (comparator(*currentIterator, value)) // Если нынешний итератор меньше, чем значение
		{
		    first = ++currentIterator; // теперь левая граница смотрит на следующий индекс за найденным 
			count -= step + 1; // Сокращаем количество элементов для поиска
		}
		else
		{
			count = step; // Оставляем лишь уполовинненый массив
		}
	}
	
	return first;
}
```

Данный алгоритм интересен тем, что движется тут 2 итератора, тот который типа `middle` и `first`, и в результате именно `first` должен указывать на нужный нам элемент.
Если немного перефразировать, то данный алгоритм либо укажет нам на самый первый из дублирующихся таргетных элементов, либо же найдет позицию в отсортированном массиве, куда можно вставить таргетный элемент.

### `std::upper_bound()`
При этом `std::upper_bound()` отличается от `std::lower_bound()` позицией аргументов компаратора:

```cpp
template<
	class ForwardIt, 
	class T = typename std::iterator_traits<ForwardIt>::value_type,
	class Compare
>
ForwardIt upper_bound(
	ForwardIt first, 
	ForwardIt last, 
	const T& value, 
	Compare comparator
)
{
	ForwardIt currentIterator; // Аналог задаваемого нами ранее элемента middle
	typename std::iterator_traits<ForwardIt>::difference_type count; // Это по факту просто число 
	typename std::iterator_traits<ForwardIt>::difference_type step; // И это тоже
	count = std::distance(first, last); // Первоначальное количество элементов, по которым будет идти поиск

	while (count > 0)
	{
		 currentIterator = first;
		 step = count / 2;
		 std::advance(currentIterator, step); // Передвигаем наш итератор на шаг (который равен половине от нынешнего количества элементов, по которым производится поиск)
		if (!comparator(value, *currentIterator)) // Если нынешнее значение более или равно нынешнему итератору
		{
		    first = ++currentIterator; // теперь левая граница смотрит на следующий индекс за найденным 
			count -= step + 1; // Сокращаем количество элементов для поиска
		}
		else
		{
			count = step; // Оставляем лишь уполовинненый массив
		}
	}
	
	return first;
}
```