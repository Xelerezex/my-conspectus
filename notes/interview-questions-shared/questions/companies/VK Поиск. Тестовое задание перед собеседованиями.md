Вопросы перед собеседованием в Поиск@Mail.ru

---

1. Предложите вариант исправления фрагмента кода:
    
    ```C++
    void process(FILE *f); // сигнатура зафиксирована
    
    void use_file(const char *name)
    {
		FILE *f = fopen(name, "w");
		process(f); // использование f, при котором может быть выброшено исключение

		fclose(f);
    }
    ```
    
    **ОТВЕТ:**
    
    Ключевая проблема в данном коде состоит в том, что исключения не обрабатываются и дескриптор открытого файла не будет отпущен.
    
    Решения данной проблемы:
    
    Обернуть код в блок `try` - `catch`:
    
    ```C++
    void process(FILE *f);
    
    void use_file(const char *name)
    {
		FILE *f = fopen(name, "w");
		
		try
        {
            process(f);
        }
        catch (...)
        {
            fclose(f);
        }
    }
    ```
    
    Так же, в дальнейшем, когда код сильнто разрастется, желательно сделать класс обертку использующую принцип _RAII_.
    
    ```C++
    class FileWrapper
    {
    
    public:
    
        FileWrapper(const char* name, const char* mode)
            : m_file{fopen(name, mode)}
        {
        }
    
        FILE* getFile () const
        {
            return m_file;
        }
    
        ~FileWrapper()
        {
            if (m_file != nullptr)
            {
                fclose(m_file);
            }
        }
    
    private:
    
        FILE* m_file;
        
    };
    ```
    
    И тогда нам не надо будет переживать, что мы забыли вызвать функцию `fclose(f);`:
    
    ```C++
    void use_file(const char *name)
    {
		FileWrapper wrapper(name, "w");
		
		try
        {
            process(wrapper.getFile());
        }
        catch (...)
        {
        }
    }
    ```
    
    ---
    
2. Предложите алгоритм для удаления дубликатов (или выбора уникальных элементов) из вектора. Оцените временную и пространственную сложность.
    
    **ОТВЕТ:**
    
    У нас дан вектор `std::vector<int> vect;` с большим количеством повторяющихся элементов.
    
    Для удаления всех дубликатов я вижу 2 варианта решения данной задачи:
    
    - Использовать `std::sort`, а потом `std::unique`:
        ```C++
        std::sort(vect.begin(), vect.end());
        vect.erase(std::unique(vect.begin(), vect.end()), vect.end());
        ```
        `std::sort` отработает за _O(N · log(N))._
        `std::vector::erase` отработает за _O(N)._
        `std::unique` в худшем случае отработает за _O(N)._
        Значит такой алгоритм работы отработает примерно за _O(N · log(N))._
    - Использовать `std::unordered_set`:
        ```C++
        std::unordered_set<int> un_set(vect.begin(), vect.end());
        vect.assign(un_set.begin(), un_set.end());
        ```
        `std::unordered_set` добавляет в себя элементы, в среднем за _O(N)_ а в худшем случае за _O(N^2)._
        `std::vector::assign`отработает за _O(N)._
        Значит весь алгоритм отработает за _O(N)_, но стоит помнить, что в худшем случае время может измениться до _O(N^2)_.
        
    
---
    
3. Сколько раз эта программа напечатает слово “hello”? Обоснуйте свой ответ.
    ```C++
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    
    int main(int argc, char *argv[])
    {
		printf("hello");
		fork();
		printf("\n");

		return 0;
    }
    ```
    
    **ОТВЕТ:**
    
    В данном коде строка `"hello"` выведется 2 раза, потому что функция `fork()` - это функция, обращающаяся к системным вызовам и создающая дочерний процесс. Дочерний процесс будет создавать полную копию адресного пространства своего родителя, а значит и `printf(”hello”);` отработает дважды - один раз в дочернем процессе, и один раз в родительском.