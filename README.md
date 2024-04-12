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
7) Затем наступает фаза commit - фаза реальной отрисовки. Она разбита на 3 части. Во время каждой из них происходит полный обход вышеупомянутого списка эффектов. BeforeMutation перед отрисовкой в DOM, commitMutation - изменения в DOM, layoutEffects - после изменения DOM. 

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
___________________________________________________________________________________________________

