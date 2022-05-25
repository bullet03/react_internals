React internals

Краткая суть

Содержание

Определения
  Реконсилиация
  FiberRoot
  RootFiber
  StateNode
  children
  childs

Мысли

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

Есть tag, который собой представляет тип fiber node. 

Fiber root создается двумя частями. RootFiber(HostRoot, tag: 3) & FiberRoot. Они повязываются через свойства FiberRoot.current = RootFiber & RootFiber.stateNode = FiberRoot.

Содержимое FiberNode больше, чем содержимое ReactElement.

Много константных вспомогатедбных значений расположено в замыкании.

Чтобы понимать код реакта, надо познакомиться с пониманием бинарных операций.
