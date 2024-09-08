---

### Ресурсы:

1. [[qtwidget]].

---

_Qt_-шная архитектура _Модель_-_Представление_ предоставляет стандартный способ для _Представлений_ манипулировать информацией, используя абстрактную модель данных. Такой подход помогает обеспечить упрощение и более стандартизированные подходы для доступа к данным.

Простые _Модели_ представляют сырые данные как таблицу элементов и позволяют _Представлениям_ доступ к этим сырым данным по средствам [[Работа с Qt на базе паттерна Модель-Представление]] систем.

Если брать более частный случай использования, то один из самых популярных будет - отображать сырые данные как дерево, позволяя каждому фрагменту данных являться родителем для набора, в табличном виде, дочерних фрагментов.

Перед тем как приступить к реализации _Модели_ дерева, стоит задуматься - данные будут поступать из какого-то внешнего источника, либо же храниться в самом классе _Модели_.

В данном примере применяется подход - хранение данных во внутренней структуре, созданной для упращения, что бы не заморачиваться с менеджерингом данных из некого внешнего источника.

---

### Дизайн и Концепция:

Структура, которую мы будем использовать для представления данных, примет форму дерева построенных из объектов типа `TreeItem`. Каждый такой объект является элементом элементом представления дерева и хранит в себе несколько колонок с данными.

---

### Структура Simple Tree Model:

![[simple-tree-item.svg]]

Данные хранятся внутри самой _Модели_, используя объекты типа `TreeItem`. Все объекты соединены в древовидную структуру на базе указателей. Поэтому, обычно, каждый объект типа `TreeItem` имеет элемент-родитель и так же может получить количество элементов-детей, для которых он сам является родителем.

Особняком идет _Корневой Элемент_ - он не может иметь никаких родителей, и иметь какие-то ссылки на данные за пределом _Модели_.

Каждый элемент типа `TreeItem` включает в себя информацию о месте, в котором он хранится в рамках Древовидной Структуры (может вернуть указатель на родителя или номер ряда, в котором хранится).

Использование структуры основанной на указателях означает, что при передаче _Индекса Модели_ в _Представление_, мы можем записать адрес соответствующего элемента в _Индекс_ ([QAbstractItemModel::createIndex](https://doc.qt.io/qt-6/qabstractitemmodel.html#createIndex)()) и извлечь его позднее при помощи [QModelIndex::internalPointer](https://doc.qt.io/qt-6/qmodelindex.html#internalPointer)(). Такая особенность позволяет упрощать запись в модель и гарантировать, что все _Индексы Модели_, которые ссылаются на какой-то элемент данных, имеет одинаковый внутренний указатель на этот элемент.

### Листинги Кода:

- Класс `TreeItem`
    - `tree_item.h`
        
        ```C++
        \#ifndef TREEITEM_H
        \#define TREEITEM_H
        
        \#include "qvector.h"
        \#include <memory>
        
        class QVariant;
        template<typename T>
        class QList;
        
        class TreeItem
        {
        
        public:
        
            explicit TreeItem(
                const QVector<QVariant>& data,
                TreeItem* pParentItem = nullptr
            );
            ~TreeItem();
        
            /*!
             * \brief Добавляем данные о ребенке этого элемента.
             *        используется при первом создании Модели и
             *        использованием не желательно при нормальной работе приложения
             * \param pChild - указатель на новый чайлд
             */
            void appendChild(TreeItem* pChild);
        
            /*!
             * \brief Позволяет модели получить чайлда данного айтема
             * \param row - номер ряда, где лежит чайлд
             * \return указатель на чайлда
             */
            TreeItem* child(int row);
            /*!
             * \brief Позволяет модели получить количество айлдов в данном айтиме
             * \return  количество чайлдов
             */
            int childCount() const;
            /*!
             * \brief Позволяет получить информацию о количестве колонок с данными,
             *        проассоциированными с этим айтемом.
             *        Данные из каждой колонки могут быть получены через вызов метода
             *        data()
             * \return  количество колонок с данным
             */
            int columnCount() const;
            /*!
             * \brief Метод позволяет вернуть данные, из какой-то колонки,
             *        проассоциированные с этой колонкой
             * \param column - номер колонки
             * \return данные
             */
            QVariant data(int column) const;
            /*!
             * \brief Метод возвращает номер ряда, в которо находится данный айтем
             * \return номер ряда
             */
            int row() const;
            /*!
             * \brief Метод возвращает указатель народителя этого айтема
             * \return указаетль на родителя
             */
            TreeItem* parentItem();
        
        private:
        
            class PrivateData;
            std::unique_ptr<PrivateData> m_pData;
        };
        
        \#endif // TREEITEM_H
        ```
        
    - `tree_item.cpp`
        
        ```C++
        \#include <tree_item.h>
        
        \#include <QDebug>
        
        class TreeItem::PrivateData
        {
        public:
        
            explicit PrivateData(const QVector<QVariant>& data, TreeItem* const pParent)
                : itemsData{ data }
                , parentItem{ pParent }
            {
            }
        
            std::vector<std::unique_ptr<TreeItem>> childItems;
            QVector<QVariant> itemsData;
        
            // Невладеющий указатель на родителя
            TreeItem* const parentItem;
        };
        
        TreeItem::TreeItem(
            const QVector<QVariant>& data,
            TreeItem* pParentItem
        )
            : m_pData{ std::make_unique<PrivateData>(data, pParentItem) }
        {
        }
        
        TreeItem::~TreeItem() = default;
        
        void TreeItem::appendChild(TreeItem *pChild)
        {
            auto temporary = std::make_unique<TreeItem>(QVector<QVariant>{});
            temporary.reset(pChild);
            if (nullptr == temporary)
            {
                qDebug() << "Error in appendChild()";
                return;
            }
        
            m_pData->childItems.push_back(std::move(temporary));
        }
        
        TreeItem *TreeItem::child(int row)
        {
            if (row < 0)
            {
                return nullptr;
            }
            if (row >= m_pData->childItems.size())
            {
                return nullptr;
            }
        
            return m_pData->childItems.at(row).get();
        }
        
        int TreeItem::childCount() const
        {
            return m_pData->childItems.size();
        }
        
        int TreeItem::columnCount() const
        {
            return m_pData->itemsData.size();
        }
        
        QVariant TreeItem::data(int column) const
        {
            if (column < 0)
            {
                return QVariant{};
            }
            if (column > m_pData->itemsData.size())
            {
                return QVariant{};
            }
        
            return m_pData->itemsData.at(column);
        }
        
        int TreeItem::row() const
        {
            if (nullptr != m_pData->parentItem)
            {
                const auto& container = m_pData->parentItem->m_pData->childItems;
                for (int index = 0; index < container.size(); ++index)
                {
                    if (this == container.at(index).get())
                    {
                        return index;
                    }
                }
            }
        
            return 0;
        }
        
        TreeItem *TreeItem::parentItem()
        {
            return m_pData->parentItem;
        }
        ```
        
- Класс `TreeModel`
    
    Данный класс очень сильно похож на всех наследников [[QAbstractItemModel]], которые предоставляют функционал _Модели_ только на чтение. Лишь реализацию конструктора и метода `setupModelData()` можно назвать не характерной для ридонли _Моделей_.
    
    - `tree_model.h`
        
        ```C++
        \#ifndef TREE_MODEL_H
        \#define TREE_MODEL_H
        
        \#include <QAbstractItemModel>
        
        \#include <memory>
        
        class TreeItem;
        
        /*!
         * \brief Класс представляющий дверовидную модель отображения данных
         */
        class TreeModel : public QAbstractItemModel
        {
            Q_OBJECT
        
        public:
        
            /*!
             * \brief Основной конструктор
             * \param data - данные, которые пойдут в Item
             * \param parent - указатель на родительский класс
             */
            explicit TreeModel(const QString &data, QObject *parent = nullptr);
            /*!
             * \brief Основной дестурктор
             */
            ~TreeModel() override;
        
            /*!
             * \brief Метод возвращает данные, хранящиеся по значению переданной роли.
             * \note    * Если нет подходящие по параметрам значение - вернется невалидный QVariant.
             * \param index - обертка над внутренним указателем на данные хранящиеся в модели.
             * \note    * Класс QModelIndex используется для размещения данных в модели данных.
             * \param role - роль, ключ по которому будет возвращаться проассоциированная
             *               с ключом информация.
             * \return найденный данный или невалидный QVariant
             */
            QVariant data(const QModelIndex &index, int role) const override;
            /*!
             * \brief Методв возвращает все флаги, примененные к данным по индексу
             * \param index - индекс, по которому будут проверяться установленные флаги
             * \return набор флагов
             */
            Qt::ItemFlags flags(const QModelIndex &index) const override;
            /*!
             * \brief Метод возвращает данные из модели по секции в хедере, с
             *        определенной ориентацией и роли.
             * \param section - секция в хедере
             * \param orientation - ориентация
             * \param role - роль
             * \return найденный данный или невалидный QVariant
             */
            QVariant headerData(
                int section,
                Qt::Orientation orientation,
                int role = Qt::DisplayRole
            ) const override;
            /*!
             * \brief Метод возвращающий Модельный Индекс элемента.
             * \param row - ряд, в котором находится элемент.
             * \param column - колонка, где находится элемент.
             * \param parent - ссылка на родительский индекс, при присутсвии.
             * \return Модельный Индекс элемента
             */
            QModelIndex index(
                int row,
                int column,
                const QModelIndex &parent = QModelIndex()
            ) const override;
            /*!
             * \brief Метод возвращает родителя по переданному индексы
             * \param index - индекс Айтема
             * \return Модельный Индекс родителя
             */
            QModelIndex parent(const QModelIndex& index) const override;
            /*!
             * \brief Метод возвращает количество рядов для Айтема с
             *        заданным родителем
             * \param parent - Модельный Индекс родителя
             * \return количество рядов
             */
            int rowCount(const QModelIndex& parent = QModelIndex()) const override;
            /*!
             * \brief Метод возвращает количество колонок для Айтема с
             *        заданным родителем
             * \param parent - Модельный Индекс родителя
             * \return количество колонок
             */
            int columnCount(const QModelIndex& parent = QModelIndex()) const override;
        
        private:
        
            /*!
             * \brief setupModelData
             * \param lines
             * \param parent
             */
            void setupModelData(const QStringList& lines, TreeItem* parent);
        
        private:
        
            class PrivateData;
            std::unique_ptr<PrivateData> m_pData;
        
        };
        
        \#endif // TREE_MODEL_H
        ```
        
    - `tree_model.cpp`
        
        ```C++
        \#include <tree_model.h>
        
        \#include <tree_item.h>
        
        \#include <QVector>
        
        class TreeModel::PrivateData
        {
        
        public:
        
            explicit PrivateData()
                : pRootItem{ nullptr }
            {
            }
        
            std::unique_ptr<TreeItem> pRootItem;
        };
        
        // NOTE: Для упрощения примера - заполнение Модели данными
        //       происходит только в конструкторе, и в дальнейшем
        //       эти данные только читаются. Поэтому такая моделт и
        //       считается РидОнли
        TreeModel::TreeModel(const QString &data, QObject *parent)
            : m_pData { std::make_unique<PrivateData>() }
        {
            // Заполняются данные только для хедера
            m_pData->pRootItem = std::make_unique<TreeItem>(
                QVector<QVariant>{tr("Title"), tr("Summary")}
            );
        
            setupModelData(data.split('\n'), m_pData->pRootItem.get());
        }
        
        TreeModel::~TreeModel() = default;
        
        QVariant TreeModel::data(
            const QModelIndex &index,
            int role
        ) const
        {
            if (!index.isValid())
            {
                return QVariant{};
            }
        
            if (Qt::DisplayRole != role)
            {
                return QVariant{};
            }
        
            TreeItem* item = static_cast<TreeItem*>(index.internalPointer());
        
            return item->data(index.column());
        }
        
        Qt::ItemFlags TreeModel::flags(
            const QModelIndex &index
        ) const
        {
            if (!index.isValid())
            {
                return Qt::NoItemFlags;
            }
        
            return QAbstractItemModel::flags(index);
        }
        
        QVariant TreeModel::headerData(
            int section,
            Qt::Orientation orientation,
            int role
        ) const
        {
            if (orientation == Qt::Horizontal && role == Qt::DisplayRole)
            {
                return m_pData->pRootItem->data(section);
            }
        
            return QVariant{};
        }
        
        // Именно через этот Метод, делегаты и Представления смогу получить доступ
        // к модели
        // NOTE: Если parent будет невалиден - отсветственность вернуть самый верхний
        //       элемент, лежит на Модели
        // NOTE: row и column - подразумевают обращение чайлду, родителя parent
        QModelIndex TreeModel::index(
            int row,
            int column,
            const QModelIndex &parent
        ) const
        {
            // Проверяем, валидные ли вообще данные были переданы в функцию
            if (!hasIndex(row, column, parent))
            {
                return QModelIndex{};
            }
        
            TreeItem *parentItem;
        
            if (!parent.isValid())
            {
                parentItem = m_pData->pRootItem.get();
            }
            else
            {
                parentItem = static_cast<TreeItem*>(parent.internalPointer());
            }
        
            TreeItem* childItem = parentItem->child(row);
            if (nullptr == childItem)
            {
                return QModelIndex{};
            }
        
            return createIndex(row, column, childItem);
        }
        
        QModelIndex TreeModel::parent(
            const QModelIndex &index
        ) const
        {
            if (!index.isValid())
            {
                return QModelIndex{};
            }
        
            TreeItem* childItem = static_cast<TreeItem*>(index.internalPointer());
            TreeItem* parentItem = childItem->parentItem();
        
            if (childItem == m_pData->pRootItem.get())
            {
                return QModelIndex{};
            }
        
            return createIndex(parentItem->row(), 0, parentItem);
        }
        
        int TreeModel::rowCount(
            const QModelIndex &parent
        ) const
        {
            TreeItem* parentItem;
            if (parent.column() > 0)
            {
                return 0;
            }
        
            if (!parent.isValid())
            {
                parentItem = m_pData->pRootItem.get();
            }
            else
            {
                parentItem = static_cast<TreeItem*>(parent.internalPointer());
            }
        
            return parentItem->childCount();
        }
        
        int TreeModel::columnCount(
            const QModelIndex &parent
        ) const
        {
            if (!parent.isValid())
            {
                return m_pData->pRootItem->columnCount();
            }
        
            return static_cast<TreeItem*>(parent.internalPointer())->columnCount();
        }
        
        \#include <QDebug>
        void TreeModel::setupModelData(
            const QStringList &lines,
            TreeItem *parent
        )
        {
            // Список (невладеющий) всех родителей
            QVector<TreeItem*> parents;
            QVector<int> indentations;
        
            parents.append(m_pData->pRootItem.get());
            indentations.append(0);
        
            int number{ 0 };
        
            while (number < lines.size())
            {
                int position{ 0 };
                // Находим длину одного слова
                while (position < lines.at(number).size())
                {
                    if (' ' != lines.at(number).at(position))
                    {
                        break;
                    }
                    ++position;
                }
        
                // Сохраняем целую строку из файла
                const QString lineData { lines.at(number).mid(position).trimmed() };
        
                if (!lineData.isEmpty())
                {
                    // Читаем данные колонки из оставшейся строки файла
                    const QStringList columnStrings {
                        lineData.split(QLatin1Char('\t'), Qt::SkipEmptyParts)
                    };
        
                    // Записываем все данные в Вариант лист
                    QVector<QVariant> columnData;
                    columnData.reserve(columnStrings.size());
                    for (const auto& columnString : columnStrings)
                    {
                        columnData.append(QVariant::fromValue(columnString));
                    }
        
                    if (position > indentations.last())
                    {
                        // Последний чайлд текущего родительского элемента теперь является
                        // новым родительским элементом если только у текущего родителя нет
                        // дочерних элементов
                        if (0 < parents.last()->childCount())
                        {
                            parents.append(parents.last()->child(parents.last()->childCount() - 1));
                            indentations.append(position);
                        }
                    }
                    else
                    {
                        // Очищаем сохраненные значения
                        while ((position < indentations.last()) && (0 < parents.size()))
                        {
                            parents.pop_back();
                            indentations.pop_back();
                        }
                    }
        
                    // Добавляем нового чайлда в родительский элемент
                    parents.last()->appendChild(new TreeItem(columnData, parents.last()));
                    qDebug() << columnData;
                }
        
                ++number;
            }
        }
        ```
        
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