---

## Ресурсы:

1. [Qt Documentation. QAbstractItemModel.](https://doc.qt.io/qt-6/qabstractitemmodel.html)

---

## Основная информация:

`QAbstractItemModel` - класс, определяющий стандартный интерфейс для Моделей элементов. Модель обязана использовать реализацию этого класса для корректного взаимодействия с другими элементами [[Работа с Qt на базе паттерна Модель-Представление]]. От этого класса не должно происходить явное создание экземпляров, приемлемо только наследование.

Класс `QAbstractItemModel` можно использовать для базового хранений данных для _Представлений_ элементов в _QML_ или же в _Виджетах Qt_.

Если стоит задача использовать в дальнейшем такие представления как _Представление Списка_ в _QML_, либо же использовать виджеты `[QListView](https://doc.qt.io/qt-6/qlistview.html)` или `[QTableView](https://doc.qt.io/qt-6/qtableview.html)` то лучше наследоваться не от этого класса а от `[QAbstractListModel](https://doc.qt.io/qt-6/qabstractlistmodel.html)` или `[QAbstractTableModel](https://doc.qt.io/qt-6/qabstracttablemodel.html)`, тк они реализует более частные случаи тех или иных моделей.

![[qabstractitemmodel-inheritance.drawio.svg]]

Лежащие в основе _Модели_ данные предоставляются для _Представлений_ и _Делегатов_ как иерархия таблиц. Если же вы не используете иерархию таблиц, то данные будут просто представляться как обычная таблица с рядами и колонками. Каждый элемент этой таблицы будет иметь уникальный индекс заданный классом `[QModelIndex](https://doc.qt.io/qt-6/qmodelindex.html)`.

![[table-model-index.svg]]

### Модельные Индексы:

Каждый элемент данных, доступный для _Модели_, обязательно имеет проассоциированный с ним _Модельный Индекс_.

Для Модельных Индексов действуют следующие правила:

- _Модельный Индекс_ можно получить при помощи функции `[index](https://doc.qt.io/qt-6/qabstractitemmodel.html#index)``()`.
- Каждый _Модельный Индекс_ может иметь родственный индекс, получаемый функцией `[sibling](https://doc.qt.io/qt-6/qabstractitemmodel.html#sibling)``()`.
- Элемент чайлд будет иметь доступ к родительскому индексу при помощи `[parent](https://doc.qt.io/qt-6/qabstractitemmodel.html#parent)``()`.

### Роли для данных:

Каждый элемент хранит в себе несколько, проассоциированых с ним, фрагментов данных. Эти фрагменты можно получить при помощи специального механизма ролей ([Qt::ItemDataRole](https://doc.qt.io/qt-6/qt.html#ItemDataRole-enum)) и функции `[data](https://doc.qt.io/qt-6/qabstractitemmodel.html#data)``()`. Данные, для всех возможных для айтема, ролей получается при вызове `[itemData](https://doc.qt.io/qt-6/qabstractitemmodel.html#itemData)``().`

Задать данные для одной конкретной роли можно методом `[setData](https://doc.qt.io/qt-6/qabstractitemmodel.html#setData)``()`. Для задачи сразу всех возможных ролей стоит использовать `[setItemData](https://doc.qt.io/qt-6/qabstractitemmodel.html#setItemData)``()` (по сути заполнение сразу всего элемента).

### Флаги:

У элементов модели можно запросить их флаги, при помощи `[flags](https://doc.qt.io/qt-6/qabstractitemmodel.html#flags)``()`, что бы проверить можно ли этот элемент переносить, выделять или манипулировать им как-то ещё.

### Чайлды Элементов:

Если элемент имеет каких-то чайлдов, то метод `[hasChildren](https://doc.qt.io/qt-6/qabstractitemmodel.html#hasChildren)``()` вернет `true` в зависимости от переданного индекса на тот или иной элемент.

### Ряды и Колонки:

У модели присутсвуют методы `[rowCount](https://doc.qt.io/qt-6/qabstractitemmodel.html#rowCount)``()` и `[columnCount](https://doc.qt.io/qt-6/qabstractitemmodel.html#columnCount)``()`, определеяющие количество рядов и колонок соотвественно. Эти методы применимы ко всем таблицам целой иерархии.

Так же новые ряды и колонки с данными могут быть добавлены через методы `[insertRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#insertRows)````` ``` `` `()` `` ``` ```` и `[insertColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#insertColumns)``()`, а удалены через `[removeRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#removeRows)``()` и `[removeColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#removeColumns)``()`.

### Основные сигналы:

Модель может выдавать сигналы при происхождении некоторых ивентов.

К примеру, сигнал `[dataChanged](https://doc.qt.io/qt-6/qabstractitemmodel.html#dataChanged)``()` задействуется при том или ином изменении абсолютно любого элемента, принадлежащего модели. Изменения в заголовках, предоставляемых моделью, приводят к активации сигнала `[headerDataChanged](https://doc.qt.io/qt-6/qabstractitemmodel.html#headerDataChanged)``()`.

Или же, если поменялась сама внутренняя структура данных у модели, к примеру произошла сортировка данных, то будет вызываться сигнал `[layoutChanged](https://doc.qt.io/qt-6/qabstractitemmodel.html#layoutChanged)``()`. Он так же будет являться маркером для всех присоединенных к _Модели Представлений_, что надо обновить отображение данных.

### Поиск:

Поиск внутри _Модели_ по данным осуществляется при помощи метода `[match](https://doc.qt.io/qt-6/qabstractitemmodel.html#match)``()`.

### Сортировка:

Если понадобилась сортировка внутренней структуры данных у модели, то следует использовать `[sort](https://doc.qt.io/qt-6/qabstractitemmodel.html#sort)``()`.

---

## Наследование от QAbstractItemModel:

В этой части описываются основные моменты, которые стоит учитывать при наследовании от класса `QAbstractItemModel`. Так же много полезной информации о наследовании лежит [[Работа с Qt на базе паттерна Модель-Представление]].

### Основные методы:

При наследовании от `QAbstractItemModel` базовым минимум, определяющем логику Модели, считается переопределение пяти чисто виртуальных метода:

1. `[index](https://doc.qt.io/qt-6/qabstractitemmodel.html#index)``()` - возвращает индекс элемента из модели, лежащий по заданной строке, столбце и родительскому индексу.
    
    При переопределении этой функции в классе-наследнике вызовите `[createIndex](https://doc.qt.io/qt-6/qabstractitemmodel.html#createIndex)``()`, чтобы сгенерировать индексы модели, которые другие компоненты _Модели/Представления_ могут использовать для ссылок на элементы в вашей _Модели_.
    
2. `[parent](https://doc.qt.io/qt-6/qabstractitemmodel.html#parent)``()` - возвращает родительский элемент _Модели_ с заданным индексом. Если у элемента нет родительского элемента, возвращается невалидный `[QModelIndex](https://doc.qt.io/qt-6/qmodelindex.html)`.
    
    Общее соглашение, используемое в _Моделях_, предоставляющих древовидные структуры данных, заключается в том, что элементы-чайлды есть только у элементов из первого столбца. В этом случае при переопределении этой функции в классе-наследнике столбец возвращаемого `[QModelIndex](https://doc.qt.io/qt-6/qmodelindex.html)` будет равен 0.
    
    При переопределении этого метода в классе-наследнике будьте осторожны и избегайте вызова методов класса `[QModelIndex](https://doc.qt.io/qt-6/qmodelindex.html)` , таких как `[QModelIndex::parent](https://doc.qt.io/qt-6/qmodelindex.html#parent)``()`, поскольку индексы, принадлежащие вашей модели, просто вызовут вашу реализацию, что приведет к бесконечной рекурсии.
    
3. `[rowCount](https://doc.qt.io/qt-6/qabstractitemmodel.html#rowCount)``()` - возвращает количество строк у переданного родительского элемента. Когда родительский элемент валиден, `[rowCount](https://doc.qt.io/qt-6/qabstractitemmodel.html#rowCount)``()` вернет количество дочерних элементов родительского элемента.
    
    _Примечание_: При реализации табличной модели функция `[rowCount](https://doc.qt.io/qt-6/qabstractitemmodel.html#rowCount)``()` должна возвращать 0, если родительский элемент валиден.
    
4. `[columnCount](https://doc.qt.io/qt-6/qabstractitemmodel.html#columnCount)``()` - возвращает количество колонок для дочерних элементов данного родительского элемента.
    
    В большинстве подклассов количество столбцов не зависит от родительского.
    
    Примечание: При реализации табличной модели, метод `[columnCount](https://doc.qt.io/qt-6/qabstractitemmodel.html#columnCount)``()` должна возвращать 0, если родительский элемент валиден.
    
5. `[data](https://doc.qt.io/qt-6/qabstractitemmodel.html#data)``()` - возвращает данные, хранящиеся в соответствии с переданной ролью для элемента, на который ссылается переданный индекс.
    
    Примечание: Если для данной роли нет возвращаемого значения, вернется невалидный (созданный по умолчанию) объект [QVariant](https://doc.qt.io/qt-6/qvariant.html).
    

### Методы связанные с чайлдами:

Рекомендация из документации: стоит переопределить метод `[hasChildren](https://doc.qt.io/qt-6/qabstractitemmodel.html#hasChildren)``()` для предоставления особого поведения _Моделей_, в которых реализация `[rowCount](https://doc.qt.io/qt-6/qabstractitemmodel.html#rowCount)``()` может быть очень дорогой.

Это позволит _Моделям_ ограничивать объем данных, запрашиваемых _Представлениями_, и может использоваться как способ реализации [отложенного заполнения (lazy population, lazy load)](https://www.geeksforgeeks.org/what-is-lazy-loading/) для данных модели.

### Включение возможности редактировать данные Модели:

Для включения возможности изменять данные внутри самой _Модели_ нужна реализация метода `[setData](https://doc.qt.io/qt-6/qabstractitemmodel.html#setData)``()` и переопределение `[flags](https://doc.qt.io/qt-6/qabstractitemmodel.html#flags)``()` (это нужно для того, что бы сто процентов быть уверенным в возможности возврата флага `ItemIsEditable`). Обязательным условием корректной отработки программы является запуск сигналов `[dataChanged](https://doc.qt.io/qt-6/qabstractitemmodel.html#dataChanged)``()` и `[headerDataChanged](https://doc.qt.io/qt-6/qabstractitemmodel.html#headerDataChanged)``()` при каждом изменении данных либо в хедере, либо в в элементах _Модели_.

Так же не стоит забывать и про возможность присваивать какие-то данные для заголовка модели: `[headerData](https://doc.qt.io/qt-6/qabstractitemmodel.html#headerData)``()` и `[setHeaderData](https://doc.qt.io/qt-6/qabstractitemmodel.html#setHeaderData)``()`.

### Создание индексов:

Пользовательские Модели обязательно должны создавать _Модельные Индексы_, которые будут использовать другие компоненты паттерна _MVC_. Для этого следует использовать уже реализованный метод `[createIndex](https://doc.qt.io/qt-6/qabstractitemmodel.html#createIndex)``()`, передавая в него ряд и колонку, где находятся данные, а так же некий идентефикатор, третьим параметром. Идетификатор может быть как внутренним указателем, так и целочисленным значением. Комбинация вышеописанных 3-х параметров обязана быть уникальной.

Пользовательские _Модели_ обычно используют эти уникальные идентификаторы в других переопределенных методах для доступа к данным айтема и получению информации о родителе или чайлдах этого элемента. Для простого примера, как работать с уникальными идентефикаторами - стоит посмотреть пример [[Simple Tree Model]].

### Роли для Данных:

Начнем разговор про _Роли_ с того, что пользовательские _Модели_ не обязаны переопределять каждую _Роль_, описанную в `[Qt::ItemDataRole](https://doc.qt.io/qt-6/qt.html#ItemDataRole-enum)`. В зависимости от формата данных, хранящихся в Модели, может быть полезно реализовывать метод `[data](https://doc.qt.io/qt-6/qabstractitemmodel.html#data)``()` только для нескольких основных _Ролей_. Большая часть _Моделей_ предоставляет, как минимум, текстовое представление своих внутренних данных по Роли `[Qt::DisplayRole](https://doc.qt.io/qt-6/qt.html#ItemDataRole-enum)` (очень хорошей практикой является использования [`Qt::EditRole`](https://doc.qt.io/qt-6/qt.html#ItemDataRole-enum) для всех данных, которые будут работать на бэкэнде программы). Хорошим тоном еще является предоставление валидных данных для таких _Ролей_, как `[Qt::ToolTipRole](https://doc.qt.io/qt-6/qt.html#ItemDataRole-enum)` и `[Qt::WhatsThisRole](https://doc.qt.io/qt-6/qt.html#ItemDataRole-enum)`. Поддержка этих ролей позволяет _Моделям_ корректно работать с базовыми _Qt Представлениями_. Но и если в итоге Модель получилась слишко специфической, то вполне здравым решением будет определить для нее только задаваемые пользователем Роли, начинающиеся с [`Qt::UserRole`](https://doc.qt.io/qt-6/qt.html#ItemDataRole-enum).

### Изменение внутреннего размера структуры данных у Модели:

Если пользователь реши создать модель, у который должен меняться размер внутренней структуры данных, то надо будет обязательно реализовать 4 метода:

- `[insertRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#insertRows)``()` - этот метод добавляет новое количество рядов в модель.
    
    В обязательном порядке прям перед самым добавлением надо вызвать метод `[beginInsertRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#beginInsertRows)``()` и сразу после завершения добавления, вызывается - `[endInsertRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#endInsertRows)``()`.
    
- `[insertColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#insertColumns)``()` - метод, позволяет добавить новое количество колонок в модель.
    
    В обязательном порядке прям перед самым добавлением надо вызвать метод `[beginInsertColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#beginInsertColumns)``()` и сразу после завершения добавления, вызывается - `[endInsertColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#endInsertColumns)``()`.
    
- `[removeRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#removeRows)``()` - метод позволяющий удалить заданное количество рядов.
    
    В обязательном порядке прям перед самым удалением надо вызвать метод `[beginRemoveRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#beginRemoveRows)``()` и сразу после завершения удаления, вызывается - `[endRemoveRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#endRemoveRows)``()`.
    
- `[removeColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#removeColumns)``()` - метод позволяющий удалить заданное количество колонок.
    
    В обязательном порядке прям перед самым удалением надо вызвать метод `[beginRemoveColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#beginRemoveColumns)``()` и сразу после завершения удаления, вызывается - `[endRemoveColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#endRemoveColumns)``()`.
    

Стоит так же упомянуть, что все 4 метода (`[beginInsertRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#beginInsertRows)``()`, `[beginInsertColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#beginInsertColumns)``()`, `[beginRemoveRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#beginRemoveRows)``()`, `[beginRemoveColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#beginRemoveColumns)``()`) имеют приватные сигналы: `[rowsAboutToBeInserted](https://doc.qt.io/qt-6/qabstractitemmodel.html#rowsAboutToBeInserted)``()`, `[columnsAboutToBeInserted](https://doc.qt.io/qt-6/qabstractitemmodel.html#columnsAboutToBeInserted)``()`, `[rowsAboutToBeRemoved](https://doc.qt.io/qt-6/qabstractitemmodel.html#rowsAboutToBeRemoved)``()`, `[columnsAboutToBeRemoved](https://doc.qt.io/qt-6/qabstractitemmodel.html#columnsAboutToBeRemoved)``()`. В официальной документации так же упоминается, что компоненты, подключенные к этому сигналу, используют его для адаптации к изменениям размеров модели. Он может быть создан только классом `QAbstractItemModel` и не может быть явно создан в коде класса-наследника (Это приватные сигналы. Они могут использоваться только в функциях `connect()`, но не могут вызываться пользователем через `emit`).

Приватные сигналы нужны для того, что бы любые компоненты, к которым эти сигналы присоединены, могли совершить какое-то действие до того, как данные будут полностью потеряны или изменены. Сокрытие пар методов `begin…()/end…()` для добавления и удаления рядов и колонок позволяет модели правильно управлять [_Постоянными Модельными Индексами_](https://doc.qt.io/qt-5/qpersistentmodelindex.html).

**Если вы хотите, чтобы работа с выборками данных внутри Модели обрабатывалась должным образом, вы должны убедиться, что вызываете эти пары функций**.

При добавлении или удалении элементов Модели, имеющих чайлдов, пары функций не нужно вызывать для дочерних элементов, потому что действует правило - Родительский элемент отвечает за удаление или добавление рядов/колонок в модель.

### Альтернативный способ добавления в Модель:

Для реализации пользовательских Моделей, которые инкрементно расширяются следует переопределить методы: `[fetchMore](https://doc.qt.io/qt-6/qabstractitemmodel.html#fetchMore)``()` и `[canFetchMore](https://doc.qt.io/qt-6/qabstractitemmodel.html#canFetchMore)``()`.

Так как переопределение метода `[fetchMore](https://doc.qt.io/qt-6/qabstractitemmodel.html#fetchMore)``()` добавляет новые ряды в Модель, перед непосредственным добавлением надо вызвать `[beginInsertRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#beginInsertRows)``()` и, когда закончится добавление - `[endInsertRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#endInsertRows)``()`.

---

## Потокобезопасность

Из-за того, что класс `QAbstractItemModel` [наследуется от](https://doc.qt.io/qt-6/threads-qobject.html#accessing-qobject-subclasses-from-other-threads) [`QObject`](https://doc.qt.io/qt-6/threads-qobject.html#accessing-qobject-subclasses-from-other-threads) - он не является [потокобезопасным](https://doc.qt.io/qt-6/qrandomgenerator.html#reentrancy-and-thread-safety). Любое API, основанное на Модели класса `QAbstractItemModel` должно вызываться только из потока в котором живет инстанс этого объекта. Если `QAbstractItemModel` соединен с каким-то _Представлением_, это значит что нигде, кроме потока _GUI_ не должно происходить вызовов его API.  
Использование дополнительного потока, для приращения или больших изменений со структурой данных самой  
_Модели_ допускается, но надо уделять особое внимание работе с данными и передачи их _Модели_. Потому что из другого потока не получится обратиться напрямую в _Модель_. Раз не выйдет обратиться напрямую, лучше рассмотреть такие варианты передачи данных, как помещение изменений в очередь, которая находится в основном потоке, и при возможность - последовательное добавление из основного потока прямо в _Модель_. Такой реализации можно дабиться при помощи [очерёдных соединений](https://doc.qt.io/qt-6/threads-qobject.html#signals-and-slots-across-threads).