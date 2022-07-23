# Моделирование зоны отбора на складах под управлением SAP EWM с целью оптимизации пробегов

## Суть проблемы со стороны бизнеса

Разговариваем только про стеллажи

### Существующие бизнес проблемы
Существует несколько проблем со стороны бизнеса, которые решает данная программа:
* Проблема интеграции нового алгоритма создания прогонов для погрузчиков на складе. 
* Проблема надежности нового алгоритма создания прогонов для погрузчиков на складе.
* Проблема  выгоды  нового  алгоритма  создания  прогонов  для погрузчиков на складе

### Описание бизнес-проблем
1. Проблемаинтеграции  нового  алгоритма  создания  прогонов  для погрузчиков  на  складе  подразумевает  под  собой  денежные  и  временные затраты  на  установку  и  настройку  нового  алгоритма  на  существующем оборудовании, форматирование данных под параметры определенного склада, обучение сотрудников, отвечающих за управление на складе.
2.  Проблема  надежности  нового  алгоритма  создания  прогонов  для погрузчиков на складе включает в себя теоретически возможную техническую неисправность в алгоритме; непредвиденные ситуации, возникшие в связи с человеческим фактором, ведущие к финансовым потерям со стороны бизнеса.
3.  Проблема  выгоды  нового  алгоритма  создания  прогонов  для погрузчиков на складеподразумевает под собой то, что результат для нового алгоритма будет проигрывать по времени по сравнению с уже существующим и использующимся алгоритмом создания прогонов.

## Суть проблемы со стороны IT

### Стратегии размещения в складские места

SAP EWM реализует стратегии размещения в складские места, посредством следующих инструментов:

- Количество и размеры областей действий.
- Сортировка складских мест по областям действий.
- Количество и размеры складских участков.
- Порядок обхода складских участков.
- Фиксированные и динамические складские места.

Инструменты выше — это основной набор для нашей задачи, но он не является полным набором функций SAP EWM.

#### Алгоритм выбора планируемого места

1. Если есть фиксированные складские места, то система пополняет их.
2. При отсутствии фиксированных складских мест для данного продукта (product) система определяет складские секции для этого же продукта.
3. Система находит доступные складские места, в которые можно положить продукт. При наличии нескольких таких складских мест, система выбирает складское место в соответствии с сортировкой по области действий.
4. В ситуации когда в выбранной секции нет свободных складских мест, система выбирает следующую наиболее приоритетную секцию и повторяет шаг 2.

#### Процесс размещения продукта в зоне коробочного пополнения

Процесс размещения продукта в зоне коробочного пополнения может происходить как из внешнего пополнения, так и из зоны хранения.

1. Система определяет складское место в соответствии с наименьшей датой срока годности.
2. При наличии нескольких складских мест с одинаковой датой срока годности, выбирается складское место, в котором хранится наименьшее количество продукта.

### Планирование обхода складских мест

SAP EWM управляет обходом складских мест, опираясь на следующие параметры и инструменты:

- Порядок сортировки складских мест в рамках области действия (Activity Area). Сортировка может задавать вручную пользователем, соответственно поддерживаются любые алгоритмы обхода.
- Количество или размеры областей действий. Складские места из разных областей действий не попадут в один складской заказ.
- Размер складского заказа (ограничение по объему, например, объему одной палеты).

#### Алгоритм планирования отбора

1. На основе заказов на исходящие поставки (ODO) система определяет потребность к отбору.
2. Система определяет отпускающие складские места на основе стратегии отбора.
    1. Наиболее популярная стратегия - это FEFO (First expired first out).
    2. Среди складских мест с одинаковой датой выхода срока годности обычно выбирается складское место с наименьшим количеством.
    3. Среди складских мест с одинаковым количеством обычно выбирается складское место согласно сортировке по области действий.
3. Система создает складские задачи, согласно выбранным складским местам.
4. В складские заказы набираются складские задачи.
    1. Складские задачи выбираются в порядке сортировки областей действий по отпускающим складским местам. Складские заказы включают в себя только те складские места, которые находятся в одной области действия.
    2. При достижении лимита по объему складской заказ считается законченным и набирается новый.
5. Полученные складские заказы являются маршрутами для ресурсов на складе.

В системе есть функционал оценки времени и пути необходимого на выполнение складского заказа под названием Расчет участка пути.

### Расчет участка пути (Travel distance calculation / TDC)

Вся зона коробочного отбора обычно представляет собой один тип склада, поэтому под сетью (network) также подразумевается один тип склада (storage type). Расчет участка пути будет происходить в рамках этой сети.

TDC включает в себя два вида передвижений:

1. Передвижение в рамках одной сети, включающее в себя передвижения по рядам (aisle) и связь между складским местом (storage bin) и сетью.
2. Передвижение вне сети.

Эвристика для TDC:

- Поиск в глубину

    Система пробует искать маршрут между указанными узлами сети в кратчайшие сроки. Здесь выполняется поиск в глубину от начального складского места до конечного. Поиск заканчивается, когда находится первый полный маршрут.

- Поиск в ширину  

    Система ищет все возможные маршруты в сети и ищет среди них кратчайший. Эта эвристика более требовательна к производительности, поэтому подходит только для небольших сетей.

- Критерий остановки  

    Параметр, ограничивающий количество узлов, которые может посетить маршрут, по отношению к маршруту, найденному поиском в глубину. Этот параметр может негативно сказаться на корректность решения и не найти корректный маршрут, однако значительно ускоряя поиск в ширину.

Как связаны эвристики с реальным миром?

TDC считает кратчайший путь между двумя складскими местами, однако из-за того, что таких складских мест в пути может быть много, то влияние эвристики TDC на общий путь будет минимально, следовательно можно опустить то, насколько оптимизирована данная эвристика.

## Моделирование

Параметры, подающиеся на вход:

- Номер склада
  - EWMWarehouse
- Порядок обхода складских мест в рамках области действия
  - ActivityArea
  - Sequence
  - EWMStorageBin
- Размер складского заказа (ограничение по объему) // описать в инструментах подробнее
- Текущий запас на складе — таблица со следующими полями
  - EWMWarehouse
  - EWMStorageType
  - EWMStorageBin
  - Product
  - ProductQuantity
  - QuantityUnit
  - EWMStockType
  - Batch
  - EWMStockOwner
  - EntitledToDisposeParty
  - ExpiredDate?
- Заказы на исходящие поставки - таблица со следующими полями:
  - EWMWarehouse
  - EWMOutboundDeliveryOrder
  - EWMOutboundDeliveryOrderItem
  - Product
  - ProductQuantity
  - QuantityUnit
  - EWMStockType
  - Batch
  - MinSLE ? Критерии выбора партии - как отдельная таблица

Параметры, получающиеся на выход:

- Складские заказы
  - EWMWarehouse
  - WarehouseOrder
  - WarehouseOrderPlannedDuration
  - WhseOrderPlanDurationTimeUnit
- Складские задачи
  - EWMWarehouse
  - WarehouseOrder
  - WarehouseTask
  - WarehouseTaskItem
  - Product
  - Batch
  - EWMStockType
  - EWMStockOwner
  - EntitledToDisposeParty
  - BaseUnit
  - TargetQuantityInBaseUnit
  - SourceStorageBin
  - DestinationStorageBin

Складские заказы включают в себя предположительное время, требующееся на выполнение заказа. Это время подсчитывается с помощью алгоритмов в SAP S/4HANA.

Ожидаемое время выполнения для каждого заказа будет просуммировано.


## Архитектура / Интеграция

### Функциональная архитектура

![Функциональная архитектура](/src/functional_schema.png)

### Пользовательский интерфейс

Пользовательский интерфейс отвечает за пользовательский ввод данных для параметров необходимых для запуска процесса моделирования. Пользовательский интерфейс включает в себя возможности просмотра предыдущих прогонов моделирования и изменения параметров подающихся на вход. После обработки пользовательского ввода, данные отправляются на серверную часть с помощью POST HTTP-запроса. 

По окончанию завершения процесса моделирования, данные передаются со стороны серверной части, обрабатываются и выводятся на экран.

Выбор пользовательского интерфейса в качестве React.js обусловлен следующими причинами:

- Высокая производительность в отношении одностраничных приложений
- Высокий уровень поддержки сообщества
- Гибкость в реализации интерфейса

### Серверная часть

Серверная часть отвечает за получение данных необходимых для моделирования со стороны пользовательского интерфейса, обрабатывает их и отправляет с помощью POST HTTP-запроса к SAP S/4HANA System REST API OData v2.

По окончанию завершения процесса моделирования, данные приходят в виде ответа на POST HTTP-запрос, сохраняются в базу данных MySQL и передаются в пользовательский интерфейс с помощью POST HTTP-запроса.

### База данных

База данных отвечает за хранение результатов предыдущих прогонов. После перезапуска приложения у пользователя должна быть возможность открыть предыдущие результаты, сравнить их.

### Сторона SAP S/4HANA System

Моделирование будет происходить в SAP S/4HANA System по следующему алгоритму:

1. Получение данных с серверной части приложения.
2. Создание складских заказов без сохранения в базе данных. Заказы хранятся в rollback области.
    1. Нужно ловить обращения SAP S/4HANA к таблицам с запасом и подменять результаты на данные полученные с серверной части приложения.
    2. Аналогично нужно подменять данные по заказам на поставки.
3. Отправка ответа на серверную часть приложения, включающего в себя созданные складские заказы.
4. Выполнение отката транзакции создания складских заказов (rollback).

## Технический дизайн

### ABAP

1. Нужно создать новый OData v2 REST API.
   1. Набор входящих/исходящих параметров аналогичен описанным в разделе моделирование
   2. Сохранить таблицу запаса в память.
   3. Вызвать создание задач путем: 
      1. Запустить создание складских задач путем вызова функции `/SCWM/TO_PREP_WHR_UI_INT` со следующими параметрами:
         1. Вход:
            1. Номер склада
            2. Позиции заказов на поставку 
         2. Выход:
            1. Складские задачи - `ET_LTAP_VB`
      2. Сохранить складские задачи в область rollback (без проведения commit) путем вызова функции `/SCWM/TO_POST` со следующими параметрами:
         1. Вход:
            1. 
         2. Выход:
            1. Таблица складских задач - `ET_LTAP_VB`
      3. Выбрать складские заказы путем вызова функции `/SCWM/WHO_SELECT` со следующими параметрами:
         1.  Вход:
             1.  `IV_LGNUM` - номер склада из поля `EWMWarehouse`, подающегося на вход в OData API.
             2.  `IT_WHO` - номера складских заказов из поля `WHO` таблицы складских задач `ET_LTAP_VB`.
         2.  Выход:
             1. Таблица складских заказов `ET_WHO`, где присутствуют поля `PLANDURA`, `UNIT_T`, отвечающие за время выполнения складского заказа и временную единицу.
      4. Заполнить тело ответа OData API из таблицы складских заказов и из таблицы складских задач.
      5. Выполнить rollback для предотвращения сохранения в базу данных SAP S/4HANA созданных складских задач и заказов.
2. Создать неявное расширение для подмены запаса в методе/классе:
   1. Прочитать таблицу запаса из памяти.

### Python

### React.js