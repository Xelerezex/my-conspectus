---

## Ресурсы:

1. [[qtwidgetsi]]

---

Данный пример является расширением функционала [[Simple Tree Model]].

В этой реализации поддерживаются:

- Элементы моделы можно изменять.
- Настраиваемые заголовки _Модели_.
- Возможость удалять/добавлять ряды и колонки (соответсвенно, позволяет добавлять новых чайлдов).

---

## Описание проекта:

Как было сказано в туториале по [[Работа с Qt на базе паттерна Модель-Представление]]_[[Работа с Qt на базе паттерна Модель-Представление]]_, **любая наследуемая базовая** _**Модель**_ должна предоставить реализации пяти основных методов:

1. `[flags](https://doc.qt.io/qt-6/qabstractitemmodel.html#flags)``()`
2. `[data](https://doc.qt.io/qt-6/qabstractitemmodel.html#data)``()`
3. `[headerData](https://doc.qt.io/qt-6/qabstractitemmodel.html#headerData)``()`
4. `[columnCount](https://doc.qt.io/qt-6/qabstractitemmodel.html#columnCount)``()`
5. `[rowCount](https://doc.qt.io/qt-6/qabstractitemmodel.html#rowCount)``()`

В дополнение к вышеописанным методам, _**Модель**_ **иерархического типа** (модель в этом примере будет как раз такой) должна реализовывать еще два метода:

1. `[index](https://doc.qt.io/qt-6/qabstractitemmodel.html#index)``()`
2. `[parent](https://doc.qt.io/qt-6/qabstractitemmodel.html#parent)``()`

Так как мы решили добавить ещё и возможность изменять внутренние элементы _Модели_, и она перестанет иметь тип только на чтение, а полный её тип будет звучать, как - **Изменяемая** **_Модель_** **иерархического типа**, надо будет озаботиться переопределением ещё двух методов:

1. `[setData](https://doc.qt.io/qt-6/qabstractitemmodel.html#setData)``()`
2. `[setHeaderData](https://doc.qt.io/qt-6/qabstractitemmodel.html#setHeaderData)``()`

А для возможности добавлять/удалять ряды и колонки в _Модель_, или другими словами - изменять размерность _Модели,_ нужны переопределения методов:

1. `[insertRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#insertRows)``()`
2. `[insertColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#insertColumns)``()`
3. `[removeRows](https://doc.qt.io/qt-6/qabstractitemmodel.html#removeRows)``()`
4. `[removeColumns](https://doc.qt.io/qt-6/qabstractitemmodel.html#removeColumns)``()`

---

## Дизайн Модели:

Ровно так же, как это было показано в примере [[Simple Tree Model]], _Модель_, которую мы будем проектировать, будет служить оберткой над коллекцией объектов класса `TreeItem`.

Класс `TreeItem` спроектирован таким образом, что бы хранить внутри себя данные для ряда элементов в _Древовидном Представлении._ Поэтому он так же должен включать в себя список значений соотвествующий данным, показанным в каждой колонке _Представления_.

Посколько [](https://doc.qt.io/qt-6/qtreeview.html)`[QTreeView](https://doc.qt.io/qt-6/qtreeview.html)` реализует _Представление Модели_ ориентированное на ряды - логичным выбором будет разработать рядо-ориентированную архитектуру для внутренний структуры данных _Модели_, которая в дальнейшем пойдет в _Представление_. Но не стоит забывать, что такой подход сделает Модель менее гибкой и подходящей для более сложных _Представлений_, чем базовые. Мы же выбираем такой подход, для упрощения разработки архитектуры и реализации классов.

---

### Взаимосвязь между внутренними элементами Модели:

![[tree-model-item-relations.svg]]

При разработке взаимосвязей между элементами Модели будет очень полезно дать им возможность обращаться к своим родителям по средствам функции `[TreeItem::parent()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treeitem-parent)`, потому что такая особенность позволит написать собственный метод _Модели_ `[parent](https://doc.qt.io/qt-6/qabstractitemmodel.html#parent)``()` гораздо легче. С такой же мотивацией стоит реализовать `[TreeItem::child()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treeitem-child)` , который может быть использован для создания [index](https://doc.qt.io/qt-6/qabstractitemmodel.html#index)() метода _Модели_.

В результате выходит, что каждый объект класса `TreeItem` будет хранить в себе информацию о:

- Своем родителе (он может быть только один в данной реализации).
- О своих дочерних айтемах (их может быть больше одного).

Эти два фактора позволят проходиться нам по всей иерархии дерева элементов.

Диаграмма выше показыват, как объекты класса `TreeItem` соеденены между собой и знают о существовании друг друга при помощи функций `[parent()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treeitem-parent)` и `[child()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treeitem-child)`. Корневой объект имеет доступ до объектов верхнего уровня **A** и **B** при помощи метода `[child()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treeitem-child)`, а до корнегового объекта **A** и **B** могут достучаться через метод `[parent()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treeitem-parent)`. Все вышесказаное работает и для объекта нижнего уровня **C**, для которого родительским будет считаться объект **B**.

---

Каждый объект класса `TreeItem` хранит все данные, которые представляют собой ряд в _Представлении_, для всех колонок этого _Предеставления_. Хранятся такие данные в поле `itemData` типа `QList<``[QVariant](https://doc.qt.io/qt-6/qvariant.html)``>`. Так как в нашей архитектуре работает правило 1-к-1 в сопоставлении ячейки колонки к элементу листа поля `itemData`, мы предоставляем довольно простую реализацию функции `[data()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treeitem-data)` для чтения записей в листе и метод `[setData()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treeitem-setdata)` для модификации. Так же как и сдругими функциями класса `TreeItem` такой подход в дальнейшем позволит без лишних проблем реализовать методы `[data](https://doc.qt.io/qt-6/qabstractitemmodel.html#data)``()` и `[setData](https://doc.qt.io/qt-6/qabstractitemmodel.html#setData)``()` в самой _Модели_, что будет хранить в себе элементы `TreeItem`.

_Корневой объект_ - это объект, который размещается самым первым в дереве иерархии, он соотвествует `_null_` _Модельному Индексу_ `[QModelIndex](https://doc.qt.io/qt-6/qmodelindex.html#QModelIndex)``()` (обычно используется для представления родительского объекта, и у рута родительский объект нулевой). Так же корневой объект не имеет физического представления ни в одном стандартном _Представлении_. Как и в дугих объектах иерархии, внутри _корневого_ хранится QList<[QVariant>](https://doc.qt.io/qt-6/qvariant.html), который используют для хранения данных о Горизонтальном Заголовке, который передается в _Представление_. Поэтому можно сказать, что метод `[setHeaderData](https://doc.qt.io/qt-6/qabstractitemmodel.html#setHeaderData)``()` является методом для установления данных исключительно в _корневой объект_.

---

### Доступ к данным через Модель:

---

![[access-data-via-model.svg]]

В диаграмме выше рассмотренном выше, фрагмент информации **a** может быть получен по средствам классического API _Модель_/_Представление_:

```C++
QVariant a model->index(0, 0, QModelIndex()).data();
```

Так как множество элементов, хранят фрагменты данных для каждой колонки в выбраном ряду, то может существовать множество _Модельных Индексов_, которые отображаются на один и тот же объект класса `TreeItem`.

К примеру, информация представленая в фрагменте **b** будет получена так:

```C++
QVariant b = model->index(1, 0, QModelIndex()).data();
```

Для получения информации о других _Модельных Индексах_ в этой же строке, что и **b**, можно было бы получить доступ через такой же базовый элементу `TreeItem`.

---

В классе _Модели_ `TreeModel` мы будем ассоцировать объекты `TreeItem` с _Модельными Индексами_ при помощи передачи указателя на каждый элемент, при создании соответсвующего Модельного Индекса при помощи метода `[QAbstractItemModel::createIndex](https://doc.qt.io/qt-6/qabstractitemmodel.html#createIndex)``()` в реализации `TreeItem::``[index()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treemodel-index)` и `TreeItem::``[parent()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treemodel-parent)`.

Мы можем получить указатели, хранящиемя таким образом, вызвав метод `[internalPointer](https://doc.qt.io/qt-6/qmodelindex.html#internalPointer)``()` у подходящего _Модельного Индекса_. В связи с этим фактом, нужно будет реализовать функцию `[getItem()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treemodel-getitem)`, которая будет делать эту работу за нас и вызывать эту функцию в методах `[data()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treemodel-data)` and `[parent()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treemodel-parent)`.

Хранить указатели на айтемы бывает очень удобно, когда мы планиуем контролировать как эти айтемы будут создаваться и уничтожаться (предполагая, что указатели, которые вернет `[internalPointer](https://doc.qt.io/qt-6/qmodelindex.html#internalPointer)``()` валидны). Но стоит помнить, что некоторые модели должны управлять элементами, которые получаются из внешних компонентов или из системы и во многих случаях не выйдет контролировать владение элементами. В таких ситуациях подход, основанный исключительно на указателях, должен быть дополнен мерами предосторожности, гарантирующими, что модель не попытается получить доступ к элементам, которые были удалены.

---

### Хранение информации во внутренней структуре Модели:

![[Untitled.svg]]

Несколько фрагментов информации хранятся как объекты `[QVariant](https://doc.qt.io/qt-6/qvariant.html)` в поле `itemData` в объекте каждого класса.

Диаграмма выше показывает как фрагменты информации, показаные в виде **a**, **b** и **c (**с маленькой буквы**)** в прошлых диаграммах, храняться внутри элементов _Модели_ во внутренней структуре, с именами **A**, **B** и **C**. Другими словами, что если взять несколько фрагментов информации из одного ряда, то это значит что они хранятся в едином айтеме.

Каждый элемент в списке соответствует фрагменту информации, предоставляемому каждым столбцом в данной строке модели.

---

Предполагается, что класс `TreeModel` будет работать с _Представлением_ `[QTreeView](https://doc.qt.io/qt-6/qtreeview.html)`, мы должны добавить ограничение, на его возможность использовать инстансы объектов `TreeItem`: каждый объект должен предоставлять одинаковое количество колонок.

Такое ограничение делает представление модели более последовательным, позволяя нам использовать _Корневой Объект_ для определения количества колонок для каждого выбранного ряда и накладывает еще одно ограничение: добавлять новые элементы только с достаточным количеством данных для колонок. В результате вставка и удаление столбцов являются трудоемкими операциями, поскольку нам нужно пройти по всему дереву, чтобы изменить каждый элемент.

Альтернативным подходом было бы спроектировать класс `TreeModel` таким образом, чтобы он усекал или расширял список данных в отдельных экземплярах `TreeItem` по мере изменения элементов данных. Однако этот "ленивый" подход к изменению размера позволил бы нам вставлять и удалять столбцы только в конце каждой строки и не позволил бы вставлять или удалять столбцы в произвольных позициях в каждой строке.

---

### Связывание элементов с использованием Модельных Индексов:

![[treeitem-child-parent-index.drawio.svg]]

Ровно так же как и в примере [[Simple Tree Model]] класс `TreeModel` должен иметь возможность брать Модельный Индекс, находить соотвествующий ему объект `TreeItem`, а также возвращать Модельные Индексы, которые соотносятся со своими родителями и чайлдами.

На диаграмме показано, что через реализацию `TreeItem::parent()` _Модель_ получает _Модельный Индекс_, соотносящийся с родителем айтема.

Указатель на элемент **C** получается из соответствующего _Модельного Индекса_ с помощью функции `[QModelIndex::internalPointer](https://doc.qt.io/qt-6/qmodelindex.html#internalPointer)``()`. Указатель был сохранен внутри индекса при его создании. Поскольку дочерний элемент содержит указатель на своего родителя, мы используем его функцию [](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treeitem-parent)`[parent()](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treeitem-parent)` [](https://doc.qt.io/qt-6/qtwidgets-itemviews-editabletreemodel-example.html#treeitem-parent)для получения указателя на элемент **B**. Индекс родительской модели создается с помощью функции `[QAbstractItemModel::createIndex](https://doc.qt.io/qt-6/qabstractitemmodel.html#createIndex)``()`, передающей указатель на элемент **B** в качестве внутреннего указателя.

---

### Листинги Кода:

- Класс `TreeItem`
    - `tree_item.h`
        
    - `tree_item.cpp`
        
- Класс `TreeModel`
    
      
    
    - `tree_model.h`
        
    - `tree_model.cpp`
        
- `MainWindow + main`
    - `main.cpp`
        
        ```C++
        \#include "main_window_wd.h"
        
        \#include <QApplication>
        
        int main(int argc, char *argv[])
        {
            QApplication application(argc, argv);
        
            MainWindowWd window;
            window.show();
        
            return application.exec();
        }
        ```
        
    - `main_window_wd.cpp`
        
        ```C++
        \#include <main_window_wd.h>
        \#include <ui_main_window_wd.h>
        
        \#include <tree_model.h>
        
        \#include <QFile>
        \#include <QDebug>
        
        class MainWindowWd::PrivateData
        {
        
        public:
        
            explicit PrivateData()
                : pUi { std::make_unique<Ui::MainWindowWd>() }
                , pTreeModel { nullptr }
            {
            }
        
            std::unique_ptr<Ui::MainWindowWd> pUi;
        
            std::unique_ptr<TreeModel> pTreeModel;
        
        };
        
        MainWindowWd::MainWindowWd(QWidget *parent)
            : QMainWindow(parent)
            , m_pData(std::make_unique<PrivateData>())
        {
            m_pData->pUi->setupUi(this);
        
            QFile file(":/default_data.txt");
            if (!file.open(QIODevice::ReadOnly))
            {
                qDebug() << "File opening error";
                return;
            }
        
            QByteArray rawData { file.readAll() };
            m_pData->pTreeModel = std::make_unique<TreeModel>(rawData);
        
            file.close();
        
            m_pData->pUi->pTreeView->setModel(m_pData->pTreeModel.get());
        
            setWindowTitle(tr("Simple Tree Model"));
        }
        
        MainWindowWd::~MainWindowWd() = default;
        ```
        
    - `main_window_wd.h`
        
        ```C++
        \#ifndef MAINWINDOW_H
        \#define MAINWINDOW_H
        
        \#include <QMainWindow>
        
        class MainWindowWd : public QMainWindow
        {
            Q_OBJECT
        
        public:
        
            explicit MainWindowWd(QWidget *parent = nullptr);
            ~MainWindowWd();
        
        private:
        
            class PrivateData;
            std::unique_ptr<PrivateData> m_pData;
        
        };
        \#endif // MAINWINDOW_H
        ```
        
- Сопутствующие файлы
    - `CmakeLists.txt`
        
        ```Makefile
        cmake_minimum_required(VERSION 3.15)
        
        project(simple-tree-example VERSION 0.1 LANGUAGES CXX)
        
        set(CMAKE_INCLUDE_CURRENT_DIR ON)
        set(CMAKE_AUTOMOC ON)
        set(CMAKE_AUTORCC ON)
        
        find_package(Qt5 REQUIRED COMPONENTS Widgets)
        
        set(HEADERS
            ./include/main_window_wd.h
            ./include/tree_model.h
            ./include/tree_item.h
        )
        
        set(SOURCES
            ./src/main.cpp
            ./src/main_window_wd.cpp
            ./src/tree_model.cpp
            ./src/tree_item.cpp
        )
        
        set(FORMS
            ./forms/main_window_wd.ui
        )
        
        set(RESOURCES
            ./resource/resources.qrc
        )
        
        qt_wrap_ui(UI_HEADERS ${FORMS})
        
        add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS} ${FORMS} ${RESOURCES})
        
        target_include_directories(${PROJECT_NAME} PUBLIC ./include)
        target_include_directories(${PROJECT_NAME} PRIVATE ./src)
        target_include_directories(${PROJECT_NAME} PRIVATE ./forms)
        target_include_directories(${PROJECT_NAME} PRIVATE ./resource)
        
        target_link_libraries(${PROJECT_NAME}
            PRIVATE
                Qt5::Widgets
        )
        ```
        
    - `main_window_wd.ui`
        
        ```Makefile
        <?xml version="1.0" encoding="UTF-8"?>
        <ui version="4.0">
         <class>MainWindowWd</class>
         <widget class="QMainWindow" name="MainWindowWd">
          <property name="geometry">
           <rect>
            <x>0</x>
            <y>0</y>
            <width>800</width>
            <height>600</height>
           </rect>
          </property>
          <property name="windowTitle">
           <string>MainWindow</string>
          </property>
          <widget class="QWidget" name="pCentralWidget">
           <layout class="QVBoxLayout" name="verticalLayout">
            <item>
             <widget class="QTreeView" name="pTreeView"/>
            </item>
           </layout>
          </widget>
          <widget class="QMenuBar" name="pMainMenuBar">
           <property name="geometry">
            <rect>
             <x>0</x>
             <y>0</y>
             <width>800</width>
             <height>21</height>
            </rect>
           </property>
          </widget>
          <widget class="QStatusBar" name="pMainStatusBar"/>
         </widget>
         <resources/>
         <connections/>
        </ui>
        ```
        
    - `resources.qrc/default_data.txt`
        
        ```Makefile
        Getting Started				How to familiarize yourself with Qt Designer
            Launching Designer			Running the Qt Designer application
            The User Interface			How to interact with Qt Designer
        
        Designing a Component			Creating a GUI for your application
            Creating a Dialog			How to create a dialog
            Composing the Dialog		Putting widgets into the dialog example
            Creating a Layout			Arranging widgets on a form
            Signal and Slot Connections		Making widget communicate with each other
        
        Using a Component in Your Application	Generating code from forms
            The Direct Approach			Using a form without any adjustments
            The Single Inheritance Approach	Subclassing a form's base class
            The Multiple Inheritance Approach	Subclassing the form itself
            Automatic Connections		Connecting widgets using a naming scheme
                A Dialog Without Auto-Connect	How to connect widgets without a naming scheme
                A Dialog With Auto-Connect	Using automatic connections
        
        Form Editing Mode			How to edit a form in Qt Designer
            Managing Forms			Loading and saving forms
            Editing a Form			Basic editing techniques
            The Property Editor			Changing widget properties
            The Object Inspector		Examining the hierarchy of objects on a form
            Layouts				Objects that arrange widgets on a form
                Applying and Breaking Layouts	Managing widgets in layouts
                Horizontal and Vertical Layouts	Standard row and column layouts
                The Grid Layout			Arranging widgets in a matrix
            Previewing Forms			Checking that the design works
        
        Using Containers			How to group widgets together
            General Features			Common container features
            Frames				QFrame
            Group Boxes				QGroupBox
            Stacked Widgets			QStackedWidget
            Tab Widgets				QTabWidget
            Toolbox Widgets			QToolBox
        
        Connection Editing Mode			Connecting widgets together with signals and slots
            Connecting Objects			Making connections in Qt Designer
            Editing Connections			Changing existing connections
        ```