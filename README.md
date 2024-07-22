# React internals

## СОДЕРЖАНИЕ
- [Описание схемы](#opisanie-shemi)
  - [Рис.1 createElement](#ris1-createElement)
  - [Рис.2 reactRoot](#ris2-creactRoot)
  - [Рис.3 render](#ris3-render)
- [Невошедшие определения](#nevoshedshie-opredeleniya)
- [Выжимка с 7km](#vizhimka-7km)

## Описание схемы

___________________________________________________________________________________________________
### Рис.1 createElement
Описание рис.1 createElement

Люди привыкли воспринимать JSX как html с возможностью встраивать JS, а на самом деле JSX это JS с расширенным синтаксисом, о чем свидетельствует даже само определение (JS Extension).

JSX представлен тегами, они похожи на html внешне, но воспринимать их как html теги - ошибка, это код который будет преобразован с помощью Babel в чистый js. Основная функция преобразованного js - React.CreateElement. Если не будет Babel, то теги не будут восприняты преобразованы корректно в функцию.

Реакт не умеет работать с JSX и HTML напрямую, только с JS. Для того, чтобы реакт начал работу ему необходимо получить на руки JS код. Эту работу выполняет за него Babel. В результате HTML или JSX распарсивается и результат парсинга передается в функцию createElement. Последняя вызывается внутри и в результате работы всегда возвращается реакт элемент.

Компонент - это вычисляемое отображение UI.
Элемент - это готовое отображение UI.

Реакт элемент - это объект с рис.1 с полями, определяющим из которых является $$typeof.

CreateElement принимает три параметра, первый обязательный, два опциональных. 1 - какой  тег(тип), 2 - пропсы, 3 - children.

В результате работы CreateElement вернется реакт объект с полями. Определяющими будут свойства key, ref, type, props, $$typeof. Свойство $$typeof - отличительное свойство реакт элемент, по которому его идентифицируют.

Приложения...
В качестве типа можно передать что угодно, не только теги. Строки и числа будут преобразованы ReactDOM в fiberNode, а булевы значения нет.

Так как children являются внутренним содержимым тега и относятся к props, то мы можем через второй параметр сразу передать третий в виде конструкции: {props: children: 'some text' }.

___________________________________________________________________________________________________
### Рис.2 reactRoot
Описание рис.2 reactRoot

reactRoot - абстрактное представление корня реакт дерева.

Имеется разделение на 2 библиотеки: React & ReactDOM. Изначально это была единая библиотека, разделение произошло позднее по причине того, что библиотеки отвечают за разное назначение. Задача React - предоставляет api для построения дерева react элементов. ReactDOM необходим для построения fiber дерева и построения DOM дерева на его основе.

Есть 4 типа деревьев. Дерево реакт узлов, дерево fiber узлов двух типов (current & workInProgress) и DOM дерево.

ReactDOM обладает множеством методов, нас прежде всего интересует метод render. Он принимает 2 аргумента: что мы рендерить и куда. Первый параметр ожидается в виде реакт элемента, второй параметр в виде корневого узла DOM, куда мы отренедерим наш реакт элемент.

В качестве первого параметра метода render можно прокинуть реакт элемент (в таком случае нам понадобится Babel для перевода его в js код), можно прокинуть объект реакт элемента с определяющим полем $$typeof, можно прокинуть тег, можно прокинуть результат выполнения функции возвращающий тег.

ReactDOM.render(<div>4565</div>, container) -> Babel
ReactDOM.render(React.createElement('div', null, 4565), container); -> createElement
ReactDOM.render({ $$typeof: ..., type: 'div', ...}}, container); -> ReactDOM

Реконсилиация - центральный процесс реакта, превращение Реакт дерева в DOM. Для этого на основании Реакт дерева построить Fiber дерево, где в процессе будут указаны все необходимые данные для отрисовки.

Fiber согласно нашим мыслям представляет собой множество понятий:
- это отдельный элемент построенный на основе React элемента нужный для того, чтобы на его основе построить DOM узел
- это дерево из отдельных fiber элементов
- это механизм/нить для того, чтобы на его основе затем построить обычное DOM дерево
- общее название архитектуры, на которой сейчас работает Реакт
  Чтобы понимать о чем идет речь, обращай внимание на контекст.

Tag (свойство Fiber) - это нумерованное обозначение типа fiberNode. В зависимости от тега будет зависесть дальнейшая обработка fiberNode.

Current Tree - ссылка на fiber дерево в предыдущем отрисованном состоянии. Оно необходимо для сравнения, чтобы понять, что именно изменилось в DOM.

Current - fiberNode, который идет в паре с WIP и при сравнение его с WIP, помогает выислить разницу в отображении.

Current & WIP связаны между собой ссылками alternate.

WorkInProgress Tree - ссылка на fiber дерево, с которым производится работа по реконсилиации в текущий момент времени.

WorkInProgress (WIP) - текущий fiberNode с которым производится текущая работа по настройке всех его полей соответствующими значениями.

Поля fiberNode - содержат всю необходимую информация для дальшейшей отрисовки DOM на экране. Пример полея fiberNode (HostRoot) ниже:
{
this.tag = tag;                     // 3
this.key = key;                     // null
this.elementType = null;
this.type = null;
this.stateNode = null;              // Fiber
this.return = null;
this.child = null;
this.sibling = null;
this.index = 0;
this.ref = null;
this.pendingProps = pendingProps;   // null
this.memoizedProps = null;  
this.updateQueue = null;
this.memoizedState = null;
this.dependencies = null;
this.mode = mode; // Effects        // 0
this.effectTag = NoEffect;          // 0
this.nextEffect = null;
this.firstEffect = null;
this.lastEffect = null;
this.expirationTime = NoWork;       // 0
this.childExpirationTime = NoWork;  //
this.alternate = null;
{
this.actualDuration = 0;
this.actualStartTime = -1;
this.selfBaseDuration = 0;
this.treeBaseDuration = 0;
}
}

FiberRoot - это механизм переключения между current & workInProgress tree. Записан как tag 0.

RootFiber/HostRoot - это корневой fiberNode, аналог document. Записан как tag 3.

Приложения...

Для мобильных устройств вместо ReactDOM будет использован React Native.

В метод render можно прокидывать cb третьим параметром, которое вызовется после того, как произойдет render.

___________________________________________________________________________________________________
### Рис.3 render
Описание рис.3 render

Необходимо посто дерево, для этого надо построить корень, на который будут насаживаться остальные элементы.

Корень создается с помощью createLegacyRoot. Результататом работы будет HostRoot. Это высшая нода Fiber, но у нее тоже есть родитель (RootFiber, tag 0), однако RootFiber не участвует в построении дерева, т.к. это механизм переключения между Current & WIP. На данный этапе мы стоим на ноде Current. В процессе создается очередь апдейтов, называемая updateQueue.

После постороения корня мы начинаем строить само дерево через updateContainer. В процессе собирает информацию, в том чсиле временную характеристику msToExpirationTime, назначение которой пока не ясно. Формула расчёта MAGIC_NUMBER_OFFSET - (ms / UNIT_SIZE | 0);

Затем updateContainer вызывает метод scheduleWork на HostRoot, в итоге вызывается функция performSyncWorkOnRoot на родителе HostRoot (т.е. это FiberRoot, tag 0), которая запускает реконсилиацию.

___________________________________________________________________________________________________
### Рис.4 reconciliation
Описание рис.4 reconciliation

Реконсилиацию можно условно разделить на 2 этапа: beginWork & completeWork. В beginWork мы полностью формируем Fiber дерево проходя от HostRoot в самый низ. В completeWork мы совершаем обратный проход от низа к руту. На обратном пути частью работы над Fiber является создание из них соответствуюших им DOM узлов.

// pushHostRootContext
// pushTopLevelContextObject
// pushHostContainer
// pushHostContext

// popHostContext
// popHostContainer
// popTopLevelContextObject

// contextFiber StackCursor
// rootInstance StackCursor
// didPerformWork StackCursor
// context StackCursor
// context StackCursor$1
___________________________________________________________________________________________________

## Вне списка (невошедшее)
есть 4 дерева
- дерево реакт элементов
- дерево fiber current
- дерево fiber workInProgress
- дерево DOM

у DOM узла есть добалвенные свойства от react. Они соедржат
- соответствующую fiber node
- события, которые были приреплены к DOM узлу самим реактом

div#root специально как-то помечается реактом. Возможно для отслеживание
его уникальности.

Перед отрисовкой приложения в div#root этот div#root вычищается от
внутреннего содержимого, включая и children и childs

Fiber root создается двумя частями. RootFiber(HostRoot, tag: 3) & FiberRoot. Они повязываются через свойства FiberRoot.current = RootFiber & RootFiber.stateNode = FiberRoot.

Содержимое FiberNode больше, чем содержимое ReactElement.

Много константных вспомогательных значений расположено в замыкании.

Чтобы понимать код реакта, надо познакомиться с пониманием бинарных операций.

В реакте часто видим применение побитовых операторов, это увеличивает производительность. Т.к. побитовые операторы малоинформативны, они именованы понятными константами на английском языке. подробнее смотреть [сюда](https://learn.javascript.ru/bitwise-operators).

В процессе debugger Chrome есть разные управляющие стрелки:
- F9 (step), шаг машинного выполнения/логики (с учетом асинхронности)
- F11 (Step into), углубиться во вложенную функцию
- Shift + F11 (Step out), выйти из вложенной функции
- F10 (Step over), идти по коду, не углубляясь в другие функции

Реакт может работать в разных режимах, они именуются контекстом. Для разных режимов заведено 6 или 7 позиций бинарного числа. 21152 - строка, где объявляются контексты. Отсутствие контекста - ноль (NoContext).

executionContext &= ~BatchedContext //  из текущего executionContext исключается BatchedContext
executionContext |= LegacyUnbatchedContext // добавляет LegacyUnbatchedContext в executionContext

Мы отслеживаем состояние момента выполнения программы через битовую маску.

update (для HostRoot) - инструкция/информация алгоритму реконсилиации для выращивание fiber дерева, которое потом првератится в DOM.

В фазе реконсилиации есть 2 процесса для каждого файбера: beginWork & completeWork.

Перед тем, как начанется фаза реконсилиации клонируется HostROOT.

Есть baseState, который с update.queue производит операции над state, update, props. Как, пока не понятно.

Update next (как и итератор) указывает на следующий update. Ссылка next самого последнего update указывает на самый первый из этой очереди.

У любого fiberNode есть по пути updateQueue.shared.pending есть ссылка на самый последний update в очереди.

!!! Требует уточнения. Со слов Абрамова pushHostRootContext нужен для того, чтобы ориентироваться, находимся ли мы в svg.

При начале beginWork над workInProgressFiber до реконсилиации его детей содержимое workInProgressFiber представляет собой содержимое currentFiber, workInProgressFiber.children => currentFiber.children
___________________________________________________________________________________________________

Fiber - набор информации. С этим набором информации нам часто приходится работать. Работа - функции (например, обновление стейта, expirationTime). Функции сами по себе тоже требуют дополнительной информации, например, приоритет, описание. Поэтому функция с ее контекстом упаковывается в объект под названием update. Вот как выглядит update. Главное поле - tag, именно он определяет, какая работа будет выполнена над fiber. 

var update = {
  expirationTime: expirationTime,
  suspenseConfig: suspenseConfig,
  tag: UpdateState,                    // 0(binary num)
  payload: null,
  callback: null,
  next: null
}

var updateQueue = {
  baseState: fiber.memoizedState,      //null
  baseQueue: null,
  shared: {
    pending: null
  },
  effects: null
};

Под тегом лежит число, которое характеризует, какие действия надо произвести с информацией текущего fiber. Таких функций, которым надо произвести работу с fiber, может быть несколько. Им надо как-то определить, в каком порядке эти update будут производить работу с fiber. Это свойство fiber, которое называется updateQueue. UppdateQueue представлен списком, элементы которого связаны между собой свойством next. По моим предположениям, кто был первым в очереди update, тот и будет выполнен раньше.

fiber.updateQueue = [upd1, upd2, upd3]
___________________________________________________________________________________________________

Общий план отрисовки.

1) Отрисовка начинается с того, что у нас есть контейнер, куда будет происхожить отрисовка (чаще всего это div с id="root"). Контейнер будет зачищен от всего внутреннего содержимого.
2) Затем нам надо предоставить то, что мы будем отрисовывать. За это отвечает ReactDOM.render(содержимое_для_отрисовки). Шаг 2 рассмотрим  подробнее.
3) Чаще всего мы предоставляем фукнциональный компонент, но также можем предоставить JSX разметку. 
4) Над аргументом метода render будет вызван метод библиотеки React createElement, который в большинстве случаев вернет react element.
5) Наступает render фаза, во время которой мы строим основу(верхушку) Fiber дерева с механизмом, к которой потом будут добавлены fiber nodes аналогичные react nodes. Фаза render заканчивается построением HostRoot. 
6) Наступает этап реконсилиации, во время которого мы сначала прохожим до самого нижнего уровня, расставляя будущие эффект тэги и прикрепляя fiber nodes. А после мы поднимаемся снизу вверху, создавая реальные DOM узлы, создавая список эффектов (реализация через firstEffect, lastEffect, nextEffect).
7) Затем наступает фаза commit - фаза реальной отрисовки. Она синхронная, разбита на 3 части. Во время каждой из них происходит полный обход вышеупомянутого списка эффектов. BeforeMutation перед отрисовкой в DOM, commitMutation - изменения в DOM, layoutEffects - после изменения DOM. Под конце снова проверяется, загрязнили ли мы корень (ensureRootIsScheduled)

___________________________________________________________________________________________________

Сопутствующие определения:
- Function - исполняемый код (Дима - исполняемый код   в виде текста предназначенный к исполнению в будущем  )
- Callback - функция, которая передается в параметры или исполняется в другой функции (Дима - будет вызван другой логикой, другой функцией)
- Job - функция или cb, которая должна быть выполнена в будущем. Запускаем ее сами (Дима - активная сохраненная функция с метаданными во время исполнения. Может также порождать, содержать метаданные)
- Task - задача, которую выполнят в будущем. Запускается автоматически (Мила - обернутый cb с метаданными)(Дима - пассивная сохраненная функция с метаданными для будущего исполнения. Как только отдаем на исполнение, она становится job)
- Listener - функция, которая будет вызвана в ответ на событие
- Effect - добавить что-то дополнительное к сущности, на которой эта функция вызвана (Дима - функция, выполняющая side effects. М.б. связано с окружением, рандомизацией)
- Это всё функции. Отличаются по связи с метаданными, моментом запуска (по событию, наступлением какой-то логики, времени)
_______________________________________________________________________________________________________

- Связь - прямое отношение двух объектов (вершин)
- Путь - последовательность связанных вершин
- Граф - состоит из множества объектов, именующихся вершинами и множеством их связей, называемыми рёбрами
- Дерево - иерархический (кроме корня у каждого узла ровно один родитель) минимально/одно (при удалении ребра граф перестает быть связным, есть только один путь) связный (между любыми двумя вершинами есть путь) ациклический граф
- Куча - полное двоичное дерево, а это значит что у каждого узла не более двух потомков и они заполняются слева направо без пропусков (выражена массивом)
- MaxHeap/MinHeap - куча  подвид дерева, у которой приоритет родительского узла больше(меньше)/равно, чем у любого из его детей
- Список - неперелинкованный набор данных
- Связный список - узлы имеют ссылки на следующий/предыдущий узлы списка
___________________________________________________________________________________________________

## Краткая выжимка с сайта [7km](https://7km.top/)

Реконсилиация - это сравнение реакт элемента(ов) с fiber(ами) current tree, в результате чего родится WIP fiber.
Реконсилиация бывает одиночная и множественная. Информация об удаляемых дочерних current fiber заносится в родительский WIP fiber. 

ReconcileSingleElement:
- параметры (WIP returnFiber, react element, currentFirstChild, lane)
- react element - один, CURRENTFirstChild -> может иметь много сиблингов
- ищется совпадение react element с child. Пока совпадение нет и еще не конец, 
child обрезаем по одному. ЕСли находим, то отрезаем сразу всех остальных
- сравнение просиходит сначала по key и type
- если оба совпали, то клонируем. Т.е. создаем новый fiber, его содержимое 
копируем с совпавшего current child. Возвращаем его, иначе создаем новый fiber, его
содержимое берем с react element


ReconcileChildrenArray:
- параметры: WIP (returnFiber), currentFirstChild, newChildren (reactEls), lanes
- задача сравнить reactEls с цепочкой currentFibers
- определяющая цепочка - массив реакт элементов, бежим по ней, все последующие действия выполняем, отталкиваясь от нее
- есть 2 for по reactEls, но счетчик между ними не сбрасывается, чтобы разделить перебор на 2 фазы: когда key совпадает и нет
- могут быть 3 ситуации:
  - key совпали и type совпали, тогда мы переиспользуем fiber 
  - key совпали, type не совпал, тогда мы не выходим из фазы совпадающих key (все еще бежим по первому циклу), но создаем новый fiber, а старый current удаляем
  - key не совпали, то завершаем первый for, переходим на второй
- суть второй фазы for (рекоснилиация после того, как перестали совпадат key):
  - создаем map из оставшихся currentFibers, где ключ - key или index, а значение сам fiber
  - продолжаем перебирать реакт элементы, пытаясь найти соответствующий им fiber из map
- если после сравнений осталось больше reactEls, создаем для них новые fibers
- если после сравнений осталось больше current fibers, то удаляем лишние deleteRemainingChildren -> deleteChild


Битовые операции: 
- представление в памяти числа в виде 32 бит, где первый бит занят под знак
- в js type Number занимает 64 бита, поэтому если обычное число с плавающей точкой перевести в 32 бита, то мы теряем часть информации (отсекаем дробную часть)
- стандартные операции: умножить ИЛИ (&), прибавить И (|), исключающее ИЛИ XOR (^), инвертирование битов (~), сдвиг влево на количество указанных битов (<<),  сдвиг вправо на количество указанных битов (>>), сдвиг вправо на количество указанных битов с заполнением нулями (>>>)
- битовая маска - наполнение смыслом двоичного числа. Плюс: быстрота. Минус: смысл неочевиден, понимание страдает
- применяется для обозначения lane, context
- lane - приоритет выполнения обработки задач (fibers)
- context - выбор режима выполнения react
- есть стандартные операции в react, для getHighestPriorityLanes, getLowestPriorityLane


Дерево:
- есть 2 пути обхода дерева: DFS/BFS. В react используется DFS- бывает в 2 видах: рекурсивный и с использованием стека с дополнительными указателями
- реализация DFS в react возможна благодаря следующим связям: return, child, sibling- DFS означает что мы идём вглубь до упора, а затем вширь/вверх
- обход дерева применяется в 2 местах: context perfomrUnitOfWork- DFS размазан между несколькими функциями в perfomrUnitOfWork


Stack и операции с ними:
- stack структура данных согласно LIFO и сопутствующие операции по ее обработке (push, pop, peek). Часто применяется в разных библиотеках (React context, execution context)
- Чаще всего технически LIFO будет реализован через массив, а может через связный список, битовую маску
- В процессе используется такое понятие, как курсор. Cursor - указатель на актуальный для программиста элемент в стеке (Api стека). В реакт context использует cursor, cursor - доступ к последнему элементу стека
- В react context в фазе реконсилиации в beginWork мы заполняем стек, в фазе completeWork опустошаем стек
- Стек в context состоит из непосредственно стек, valueStack (стек) -> valueCursor ({current: ...}) -> value (значение из контекста)
- каждый fiber записывает свою техническую информацию в общий стек (любую, строку, стандарт, объект, див, число...)
- cursor имеет стандартную форму в виде объекта с полем current. cursor в реакте существует несолько разных видов. Они не обязательно указывают на вершину стека, являются глобальными переменными реакта с актуальной информацией для работы реакта

HeapSort:
- бинарное дерево (куча), чтобы обеспечить максимальное быстрое поиск/изъятие самой приоритетной задачи за O(1), потому что она на верхушке дерева
- представлена в виде массива, где значение детей всегда меньше чем родителей. Индексы детей всегда одинаковы и в 2 раза больше индекса родителя по формуле (2n + 1), за счет этого достигается оптимизация обхода дерева
- Обход дерева при перетряске имеет сложность O(n*logn)
- операции: взятие, вставки, перетряхивания. Взятие - O(1), т.к. берется самый приоритетный fiber, после этого последний элемент ставится на место первого и происходит перетряска дерева сверху вниз. Вставка - новый элемент в самый конец, после чего происходит перетряхивание всего дерева снизу вверх. Перетряхивание дерева - может быть в двух видах: сверху вниз (более сложный, т.к. сравнение происходит с 2мя детьми) и снизу вверх (более простой, т.к. сравнение происходит только с родителем)

LinkedList:
- структура, где один элемент ссылается на другой посредством ссылки next
- выгоден для вставки, удаления O(1). Сложен для поиска O(n)
- бывает нескольких видов: однонаправленный, двунаправленный, кольцевой
- доступ к списку осуществляется через указатель, т.е. какой-то элемент списка (чаще всего первый)
- в реакт используются следующие операции со списками. Важно концептуально понимать их: 
  - вставка элемента в конец кольцевого списка. В реакте кольцевые списки хранятся указателем на хвост, а не начало.
  - merge кольцевого и линейного списка => линейный. У кольцевого рвем связь последний/первый и соеднияем хвост линейного с началом кольцевого
  - merge кольцевого c кольцевым => кольцевой. Рвём связи у двух кольцевых, превращая их в линейные, а потом оба этим списка превратим в один кольцевой

Context:
- context - это система, которая состоит из самого context, Provider & Consumer
- хранит в себе value, Provider & Consumer. Последние ссылаются на context, потому что они работают с ним. Provider записывает в него, Consumer получает инфу из него
- Provider представлен react el, с соответствующим ему работе в beginWork
- Provider занимается отслеживанием изменений, в случае если они поменялись, он уведомляет Consumer
- Есть 3 вида Consumers: useContext, context.Consumer, static context property (классовый компонент)
- Consumer, считывая context, сохраняет его себе в свойство dependencies. Когда в Provider изменится значение, он сможет найти соответствующий сохраненный context нашего Consumer из списка dependencies, чтобы пометить именно этот Consumer измененным
- Перед каждой вычиткой Consumer, react зануляет dependencies, потому что они в любом случае будут обновлены при считывании

- Provider получая новое значение, не затирает старое, а кладет его в stack
- Нужно уведомить всех подписчиков (Consumers) об изменениях. Суть уведомления в том, чтобы Consumers были загрязнены и запланирован rerender
- С помощью DFS алгоритма мы спускаемся вниз в поисках наших Consumers, загрязняя только найденные Consumers (merge lanes) 
- Поднимаясь на обратном пути, загрязняем lanes всех родителей либо до workInProgress (Provider), либо до FiberRoot
- Не путать с complete. Он поднимает уже загрязенные fibers, а не lanes

- Provider в целом везде, что в context, что в redux нужен не только для того, чтобы прокидывать инфу ниже, а чтобы вызывать react rerender consumers с измененным value. Для consumers не нужен Provider чтобы считать value. Consumer сам спокойно считывает value напрямую из импортированного объекта context, т.е. в useContext кладется импортированный context
- почему react не позволено из импортированного объекта context считывать данные не только в отношениях ancestor => child, но и где угодно, например, между siblings. Это привело бы к проверке полного fiber tree всего кадра, что теряло бы смысл в системе context в целом, легче просто перерисовать кадр

Макропакеты реакта:
- react, который включает самые основные функции по работе с реакт элементами (создание, клонирование, работа с minHeap)
- reactDOM, который для визуализации и для работы с браузером
- react-reconciler, который соединяет react & reactDOM
- scheduler, который дает возможность поместить реакт в event loop и чтобы не лагало

Общий процесс работы реакта (через эти макропакеты):
- есть 2 возможности для начала запуска реакта (trigger phase), первая отрисовка или реакция на события
- в первом и во втором случае у нас должна пройти одна большая тяжелая функция (реконсилиация)
- реконсилиация может выполниться за раз в синхронном режим (браузер может лагнуть), либо в  асинхронном режим разбив на части и выполняя их в отдельных макротасках
- сосредоточимся на асинхронном режиме. важно учесть следующие нюансы:
  - большая функция внутри себя д.б. разбита на части, чтобы можно было ее подробить
  - взаимдоействие с event loop просиходит через scheduler
  - есть 3 разных типа задач: eventLoop task, scheduler task, react task
    - eventLoop task - макрозадача
    - scheduler task - task из minHeap
    - react task - обсчет одного fiber
- ??? если это асинхронный режим, то мы должны прерывать работу нашей тяжелой функции, через промежутки прерываться, чтобы дать браузеру вздохнуть. Мы дробим нашу большую функцию на части и с помощью внутреннего while выполняем маленькие шаги, пока шаг внешнего while дает нам такую возможность. Scheduler запускает большую функцию обновления

Ref:
- чаще всего используется для выхода из системы React
- перерендер fiber не сбрасывает состояние ref
- программист может в ref хранить любую информацию(управляет сам программисте), но чаще всего его связывают с DOMel(полагается на автоматику react)
- имеет структуру ref.current = {}
- ref может быть функциональным, рассчитывать его будет сама система реакта в момент вызова, передав в качестве первого параметра этой функции необходимый DOMel
- ref устанавливает значение DOM элемента на фазе commit, после append DOM элемента
- есть 2 способа обновить DOMel в ref: mount/unmount DOMel, указать другое значение в ref
- обновление идет через null, т.е. сначала мы зануляем предыдущее значение ref, потом устанавливаем новое
- нет update, есть только цикл unset, set. ref вызывается в разные моменты жизненного цикла при mount/unmount и update.  Будьте осторожны при использовании ref в cleanup useLayoutEffect (м.б. null)
- ref является динамическим js выражением (conditional, последний элемент списка и т.д.)
- если на каждом кадре значение атрибута ref меняется (стрелочная функция), то это приводит к обновлению ref через null
- связать/синхронизировать child/local и parent/external ref - называется merge рефов
- существуют фокусы, когда знаением ref является хук изменения состояния. Это несет в себе плюсы, но это неочевидно, поэтому использовать не будем. Пока не доросли =)
- ![ref lifecycle](/img/ref_lifecycle.png)

Strict Mode:
- вызывает дважды синхронно функциональный компонент, вокруг которого он обернут
- отрабатывает всегда на любом компоненте во время жизни всего приложения в dev mode в отношении вызова функции самого компонента, но не его эффектов
- хуки useEffect, useLayoutEffect при первоначальном маунте вызываются дважды синхронно, включая эффект очистки. При перерендерах хуки вызываются один раз
- помогает найти ошибки, например, с забытой функцией отписки. Из-за особого двойного вызова компонента и его эффектов могут пойти ошибки
- для предотвращения асинхронных запросов программист должен сам найти способ предупредить их повторную отправку/кеширование

Portal:
- нужны для того, чтобы отобразить содержимое компонента в другом месте DOM. Важно, в системе JSX ничего не меняется и элемент, например, получит родительскийй контекст, т.к. она на месте 
- портал создается функцией createPortal, которая принимает JSX, который надо перенести и DOM элемент, куда это надо перенести. Место определяет сам программист

Запуск приложения/формирование корня:
- в рамках развития реакта старт менялся (был legacy blocking, стал concurrent)
- сначала формируется корень приложения, он состоит из трех объектов: интерфейсный reactRoot (должен поддерживать методы монтирования/размонтирования), fiberRoot (управляющий механизм, переключение между current/WIP), hostRoot (куда будет строиться fiber волокно)
- порядок рождения этих объектов: user рождает reactRoot, reactRoot рождает fiberRoot, fiberRoot рождает hostRoot
- также в стартовую систему включен div#root и корневой react el <App/>
- в результате запустится функция updateContainer (только при первом запуске), она запустит scheduleUpdateOnfiber. Последняя будет запускаться всегда, если мы хотим обновить дерево (провести reconciliation)

SyntheticEvents System:
- react ловит события на div#root, а не на document как было в ранних версиях
- react нужна своя система событий, чтобы прогонять события по fiber дереву (выгодно потому что универсально независимо от host среды, события распространяемое в DOM tree может отличаться от пути распространения события в fiber tree => Portal)
- создание listener основывается на заранее подготовленному списку событий (списков событий 2: delegate, nondelegate)
- listener это реактовский слушатель нативных событий. На каждое имя события создаётся свой вариант react listeners (например, onClick, onClickCapture)
- созданные react listeners крепятся к div#root через обычный addEventListener с уечтом следующих флагов: +/- passive, +/- bubbling. Bubbling (delegate events), capture (delegate + nondelegate events), флаг passive (отсекается ли e.preventDefault)
- dispatch vs emit. Dispatch - отправить адресату, более широкое понятие, аргумент - eventObject. Emit - сгенерировать/выпустить, уведомить что событие произошло, аргумент emit - string/eventName
- plugin/ plug-in, подключить расширение, которое расширит основной функционал, не изменяя его структуры
- задача реакта взять native event, сгруппировать его по какому-то принципу и через extract events создать react event, положить в dispatchQueue созданное синтетическое событие, связанные с ним listeners
- processDispatchQueue прогонит все события, в зависимоссти от фазы capture/bubbling (сначала, либо с конца списка)
- в react есть eventPluginSystem, которая классифицирует events, а значит и алгоритм их обработки по типу. SimpleEventPlugin обрабатывает базовые events, такие как click. Другие pluginEventSystem обрабатывают более сложные случаи, например, enterLeaveEventPlugin, selectEventPlugin, ...
- реакт собирает все события в listeners, пробегаясь наверх. Например, могут быть несколько onClick и они все будут в listeners onClick
- создаются синтетические события, которые соединуются с listeners в единую структуру
- собранные события обрабатываются в 2 вложенных цикла: внешний перебирает различные типы events (т.е. отельно обрабатываются onClick, onChange,...), внутренний перебирает listeners прикрепленных к кажому типу событий (т.е. на один event м.б. несколько прикрепленных listeners)
- onClick, onChange,... являются props, их значения могут меняться с течением component lifecycle
- обработчики событий сработают только для HostComponent (не для Function, Class, ...), на это стоит проверка
- в 16 и 17 версии react все SyntheticEvents всплывали, даже scroll, что приводило к проблемам. С 17 версии этот баг поправили только для scroll. В будущем хотели поправить для всех nonDelegateEvents
- processDispatchSingle прогонит все listeners на данном событии. Если встречается stopPropagation и react видит, что мы перехожим/всплываем к следующему fiber, то прекращаем всплытие
- onChange react отличается от нативного тем, что callback отрабатывает сразу при введении изменений в input, в то время как у браузера callback работает лишь при blur
- selectionChange у react повешено на document, не на div#root
- ![events](/img/events.png)

- Trigger event phase (dispatchEvent/user)
- dispatchEvent - почти равнозначен наступлению события, как-то click,change,... Отличия dispatchEvent в том, что он синхронный и есть моменты с безопасностью. Бразуер по default различает события и он по-особому обрабатывает события, вызванные искуственно. Кроме клика искусвенные события работают так будет у них стоит preventDefault [ссылка](https://github.com/jsdom/jsdom/issues/3117)
- основная идея в том, чтобы пробежать строго по цепочке/пути/схеме и вызвать все связанные обработчики
- сначала собирается путь/схему распространения событий. Путь - это выявленная цепочка узлов, которые надо обойти, чтобы собрать обработчики соответствующие распространению события
- затем по этой схеме вниз будет от window выполняться!!! фаза capturing, а после фаза bubbling
- выполнять фазу capturing, например, означает что мы будем идти по схеме (не по всем узлам всего дерева) сверху вниз, выполнять по очереди все связанные обработчики
- затем все события отстреливаются по принципу сверху вниз, снизу вверх. Т.к. у браузера нет деления по capture типу событий, как у реакта, на одно событие повесятся как события со всплытием, так и без
- если нет прикрепленных listeners, то dispathEvent отработает вхолостую


Обобщенная схема работы react:
- trigger phase запускает scheduleUpdateOnFiber => ensureRootIsScheduled => scheduleCallback => performWorkOnRoot(Sync/Async) => begin/complete => commit => DOMrender
- scheduleUpdateOnFiber загрязняет fiber той работой (update), которую надо с ним сделать. Технически это выражается разными флагами, в том числе lane как метка важности обработки fiber. Например, setState => dispatchAction => scheduleUpdateOnFiber
- ensureRootIsScheduled - убеждается, что на root повешена работа. Либо lane(загрязнение) есть и тогда запусти планировщик, либо noLanes и тогда return. Если текущая задача уже стоит в scheduler с таким же приоритетом, то также return. А если приоритет загрязнения выше уже стоящей задачи в scheduler, то в scheduler отменить старую задачу и поставить новую с новым приоритетом. callbackNode - это сохраненная таска scheduler'а которая привязана к текущему root (когда отменяем старую задачу, CallbackNode.callback = null)
- scheduleCallback это не часть react, это планировщик:
  1)создать таску на основе callbackNode
  2)поставить таску в eventLoop браузера используя асинхронный api среды
  вернуть здесь и сейчас таску
- performWorkOnRoot входная точка во внутренний цикл react, которая его запускает. Стартует с root, должен раскрутить всё дерево до реального кадра (begin/complete + commit). В результате работы может вернуть либо null, либо функцию performWorkOnRoot с нового fiber, на котором закончилась обработка, т.к. performWorkOnRoot в async режиме прерываемый. Если вернулся null, значит задача по построению кадра выполнена полностью и задача снимается с самого scheduler из minHeap (taskQueue)
- ...begin/complete => commit => DOMrender

Lanes:
- тот кто тригерит fiber на изменение, тот и выставляет lane
- есть куча работы (у react - fibers, у scheduler - таски), которую надо выполнить, ее надо приоритезировать (потому что runtime js однопоточный). Приоритезировать - распределить задачи по времени (что-то позже, что-то раньше). Как мы определяем, какая задача выполняется раньше, какая позже? 
- fiber ориентируются на lane, таски ориентируются на expirationTime, это НЕ одно и тоже!!!
- lane метка важности, выраженная битовой маской
- expirationTime - реальное время, когда ТРЫНДЕЦ как важно взять в работу таску, не путать со startTime - время, когда ПРОСТО надо взять в работу таску
- существует связь между lane и expirationTime, мы перегоняем lane от fibers => в expirationTime тасок. Нюансы: lanes бывает 32 видов, а expirationTime - одно время. Связь осуществлятся через константные приоритеты т.е. lanes => priorities => startTime + DELTA (на основании priorities) => expirationTime
- у fiber существуют 2 вида lanes: собственные и сумма lanes всех его children. При подъеме lanes с дочернего fiber на родительский fiber, мы у дочернего складываем (merge) его собственные lanes с его дочерними, а потом их сумму записываем как родительский.childLanes. Таким образом все lanes всех узлов поднимаются до корня
- процесс подъема lanes происходит в triggerPhase (в scheduleUpdateOnFiber)
___________________________________________________________________________________________________

