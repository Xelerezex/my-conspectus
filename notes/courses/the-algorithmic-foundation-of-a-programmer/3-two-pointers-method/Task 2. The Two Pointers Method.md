
---
## Задача № 2

- Дано два отсортированных массива целых чисел
- Найти в них пару элементов, сумма которых равна `X`
- Пример
	- `A = {0, 2, 5}, B = {-3, -1, 2}, X = 4`
	- Ответ: пара индексов `(1; 2)`

Идея решения этой задачи:
	![[algo-foundation-two-pointers-10.png]]
	 Рассмотрим вот такой кейс, что сумма двух значений больше, чем нужная нам. В таком кейсе мы вроде бы можем уменьшить `i`, а так же и `j` а может быть и оба индекса. Пока что не совсем понятно как праильно поступить.
	 Тогда попробуем зайти с другого конца: и поставить один указатель в начало, второй указатель в конец: ![[algo-foundation-two-pointers-11.png]]
	 Что нам надо делать в этой ситуации, когда `A[i] + B[i] > X`? Конечно же уменьшить `j`, потому что эти два массива отсортированы и по определению, если уменьшить один индекс - общая сумма точно уменьшится или хотя бы останется такой же.
	 Получаем: ![[algo-foundation-two-pointers-12.png]]
	 Все еще такой же кейс, поэтому сместим `j` еще раз:![[algo-foundation-two-pointers-13.png]]
	 Теперь сумма стала меньше, чем таргетное `X`, уменьшать `j` больше нет смысла, поэтому мы увеличим `i`: ![[algo-foundation-two-pointers-14.png]]
	 Сумма теперь больше `X` - значит уменьшаем `j`: ![[algo-foundation-two-pointers-15.png]]
	 Теперь сумма меньше искомой, а значит снова увеличиваем `i`: ![[algo-foundation-two-pointers-16.png]]
	 Сумма все ещё меньше искомой, значит снова увеличиваем `i`:  ![[algo-foundation-two-pointers-17.png]]
	 И таким образом мы нашли искомую сумму!

Теперь мы оформим данный алгоритм словами:
- Ставим указатель `i` в начало первого массива, а указатель `j` в конец второго массива.
- Пока текущая сумма больше искомой, уменьшаем `j`.
- Увеличиваем `i` на 1 и возвращаемся на прошлый шаг.

### Реализация

```cpp
struct Indexes
{
    int first { 0 };
    int second { 0 };
};

Indexes TheTwoPointersMethod(
    const std::vector<int>& firstSortedVector,
    const std::vector<int>& secondSortedVector,
    int targetNumber
)
{
    Indexes result;
    
    int right = secondSortedVector.size() - 1;
    for (int left = 0; left < firstSortedVector.size(); ++left)
    {
        while (right >= 0 
            && firstSortedVector[left] + secondSortedVector[right] > targetNumber
        )
        {
            right -= 1;
        }
        
        if (right >= 0 && targetNumber == firstSortedVector[left] + secondSortedVector[right])
        {
            result.first = left + 1;
            result.second = right + 1;
            break;
        }
    }
    
    return result;
}
```