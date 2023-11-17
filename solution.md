#изменения в коде
1. Добавил новый класс Configuration
1.1. Клас будет считывать конфигурации и хранить в себе необходимые настройки (и их начальные значения) для других обектов
1.2. Далее в другие объекты будем передавать этот класс (или его структуры) с необходимыми для конкретного объекта настройками
Это, на мой взгляд, упростит реализацию необходимых фич и упростит добавление новых фич в будущем
Сам класс представлен в файле configuration.h
1.3. В этот класс необходимо добавить функцию чтения файла конфигурации и заполнения структур настроек

2. Изменения в классе Block
2.1. Избавился от глобальных переменных и перенес их в класс (и using тоже). Теперь этот объект принимает свои настройки в конструкторе и инициаизируется
Это позволит задать размеры блока и их формы, а так же, в будущем создать несколько объектов данного типа с разными размерами и формами
2.2. Новая реализация не позволяет содавать статические массивы, поэтому я перешел на динамический - вектор

3. Изменения в классе Board
3.1. Аналогично с классом Block избавился от глобальных статических переменных, замениы их динамическими переменными класса
и обавил зависимость и инициализацию в соответствии с настройками из файла конфигурации
Это позволит в будующем управлять размерами игрового поля из файла конфигурации

4. Изменения в классе Render
4.1. Данный класс теперь не просто выводит состояние игрового поля в консоль, а хранит в себе для дальнейших манипуляций
4.2. Принимает настройки из Configuration, и производит инициализацию контейнера для хранения игрового поля и положения фигур
Данное решение позволяет совершать любые манипуляции с текущим состоянием игрового поля (записать в файл, отправить куда-нибудь и т.д.)

5. Изменения в классе Games
5.1. Конструктор принимает настройки конфигурации и инициализируется в соответствии с ним (так же инициалиирует фигуру)
Это позволит в будующем через файл конфигурации задавать специализированные настройки иры
5.2. defaultPosition перенес сюда т.к., на мой взглят, именно игровой движок решает где в начале расположить фигуру
Это позволит в будующем поменять начальное положение фигуры на поле (например задав через файл конфигурации)

6. Другие изменения:
6.1. Классы Block, Board, Game, Render я переименоваk со struct в class т.к. это не пассивные объекты, они имеют функциональность и они будут развиваться по мере необходимости


#Поддержка фич
1. Изменения описанные в пункте 1 и 2 раздела "изменения в коде" позволяют реализовать Фичу 1 (управление формой фигур)
класс Configuration считывает необходимые настройки для фигуры, а затем они передаются в Block, который инициализируется в соответствии с заданными настройками
1.1. Для реализации данной фичи необходимо:
- В класс Configuration добавить функцию чтения файла конфигурации и заполнения структур настроек для фигур (BlockConfig).

2. Изменения описанные в пункте 1 и 4 позволяют реализовать фичу 2 (запись положения на игровом поле в файл и трансляция событий игры через сетевые соединения)
класс Configuration считывает необходимые настройки для рендеринга и передает их в класс Render, который в зависимости от этих настроек совершает манипуляции над игровым полем
2.1. Для реализации данной фичи необходимо:
- В класс Configuration добавить функцию чтения файла конфигурации и заполнения структур настроек рендеринга
- реализовать функции записи состояния игры в файл и трансляции через интернет

3. Изменения описанные в пункте 1 и 5 позволяют реализовать фичу 3 (управление логикой удаления заполненных линий)
класс Configuration считывает соответствующую настройку и она передается в конструктор класса Game (enum RemoveLinesLogic)
3.1. Метод nextStep переработан так, что бы в зависимости от настройки вызвать необходимый метод для удаления линий
3.2. Для полной реализации данной фичи необходимо реализовать функции удаляющие линии по выбранной логике: removeThreeFullLines, removeNotRemove, removeNinetyPercent


#Другие предложения по изменению
1. Классы Board, Game я бы лучше описал в двух файлах: .h и .cpp, так они будут более читаемыми
2. Файл moves выгледит лишним. Эти методы относятся к логике игры и врятли их будет использовать кто-то еще. Я бы предложил их перенести в класс Game
3. Некоторые методы класса принимают переменные имеющие тоже название, что и объект класса, это может путать
4. std::pair - не очень удобен, когда проект разрастется, может забыться что он значит и чточат в нем first и second. Я бы предложил заменить его на структуру:
struct BlockPosition
{
    int y;
    int x;
};
5. Сейчас при любом передвежении блока создается новый объект (Block) с новым положением, далее он проверяется на корректность положения и либо остается и удаляется старый блок, либо удаляется.
Создание нового динамического объекта занимает прилично времени, на мой взгляд, лучше менять имеющийся, а не создавать каждый раз новый (я про функции nextStep и tryNextStep в классе Game).
Например:
- запомнить текущее положение блока
- изменить его положение
- проверить, что новое положение валидно
- если валидно, то оставить переменную block с новыми значениями
- если нет, вернуть как было
