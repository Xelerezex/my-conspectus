
---
### Ресурсы:

1. [Qt Documentation. Model View Programming](https://doc.qt.io/qt-6/model-view-programming.html#models).
---
[[QAbstractItemModel]]
!!!INSERT OTHER TEXT HERE!!!

---

### Индексы Модели.

Для гарантии того, что данные хранятся отдельно от способа доступа к ним, была представлена концепция - _Индекс Модели_.  
Каждому фрагменту данных, который может быть получен из  
_Модели,_ соответствует _Индекс_. _Представления_ и _Делегаты_ используют эти _Индексы_ для запроса фрагментов данных, применяемых при последующем изображении.

В результате чего, только _Модель_ должна знать как получать данные, а так же их тип, который может определяться довольно обобщенно. Каждый _Индекс Модели_ хранит в себе указатель на _Модель_, что создала его. Эта особенность исключает путаницу при работе с более, чем одной _Моделью_.

```C++
QAbstractItemModel* model = index.model();
```

_Индексы Модели_ предоставляют **временные** ссылки на фрагменты данных и могут быть использованы для извлечения или модификации по средствам _Модели_. Стоит учитывать, что сама _Модель_ может реорганизовывать свою внутреннюю структуру время от времени и _Индексы Модели_ могут стать полностью невалидными, из-за этой особенности их не стоит подолгу хранить.

Если логика работы программы подразумевает длительное хранение ссылки на фрагмент данных, то стоит использовать _Постоянный Модельный Индекс_, потому что ссылка, предоставляемая таким индексом, будет обновлять самой Моделью и будет актуальной.

Временные _Индексы Модели_ создаются при помощи класса `[QModelIndex](https://doc.qt.io/qt-6/qmodelindex.html)` и _Постоянные Индексы Модели_ создаются классом `[QPersistentModelIndex](https://doc.qt.io/qt-6/qpersistentmodelindex.html)`.

Для получения Индекса Модели, который ссылается на какой-нибудь фрагмент данных следует передать три основных свойства:

1. Номер ряда.
2. Номер колонки.
3. Индекс модели от родительского элемента.

---

!!!INSERT OTHER TEXT HERE!!!

  

  

  

!!!INSERT OTHER TEXT HERE!!!

## Наследование от базовых классов Модели:

!!!INSERT OTHER TEXT HERE!!!