
---
Собеседование было очень странное, в камере на той стороне были какие-то скуфы, все сидели в офесе и видно было, что мебель херовая, как и компы. Оч странные впечатлени от собеса - компанию я даже и не помню

---

Комментариями написаны вопросы, которые удалось вспомнить:

```cpp
uint8_t first = 192;
uint8_t second = 168;
uint8_t third = 1;
uint8_t fourth = 2;

// Как записать 4 числа uint8_t в int.
int Ip |= 192 << 32;

char shift = 255;

int* pIp = &Ip;

*(pIp + shift) = 192;
*(pIp + shift + shift) = 168;

void f()
{ 
  // Как задать new char[2] через unique_ptr?
  // Какая происходит ошибка при char* x1 = new char[2], если объект не создался
  std::unique_ptr<char> x1 = std::make_unique(new char[2]);
  std::unique_ptr<char> x2 = std::make_unique(new char[2]);
  
  char* x1 = new char[2];
  char* x2 = new char[2];

    
  Do(x1,x2);
  delete[] x1; // [3] -> 0x0 + 3
  delete[] x2;
}
*int(void) func;

Как будет выглядеть указатель на функцию?
int f();
auto f1 = [](){ return 0; };
```