# Стратегия (Strategy)

**Шаблон Strategy** или шаблон проектирования Стратегия относится к поведенческим шаблонам проектирования. Его задача - выделить схожие алгоритмы, решающие конкретную задачу. Реализация алгоритмов выносится в отдельные классы и предоставляется возможность выбирать алгоритмы во время выполнения программы.

Шаблон дает возможность в процессе выполнения выбрать стратегию (алгоритм, инструмент, подход) решения задачи.

## Проблема

Дочерние классы наследуют методы и свойства родительских классов, при условии, что они относятся к открытым или защищенным элементам. Этим фактом можно воспользоваться для создания дочерних классов, обладающих особыми функциональными возможностями.

На **рис. 1** приведен пример наследования, демонстрируемого с помощью диаграммы UML:

![Image alt](https://github.com/IlnarAhm/php-design-patterns/raw/main/Strategy/img/UML-1.png)

<figcaption align="center"><b>Puc. 1.</b> <i>Родительский класс и два дочерних класса</i></figcaption>
<br/><br/>

Абстрактный класс Lesson на рис. 1 моделирует обучение в коллед- же и определяет абстрактные методы _cost()_ и _chargeType()_ . На диаграмме показаны два реализующих их класса, _FixedPriceLesson_ и _TimedPriceLesson_, которые обеспечивают разные механизмы оплаты занятий.

Но что произойдет, если потребуется внедрить новый ряд специализаций? Допустим, потребуется работать с такими элементами, как лекции и семинары. Они подразумевают разные способы регистрации учащихся и создания рабочих материалов к занятиям, для них нужны отдельные классы. Мне нужно обрабатывать стратегии оплаты и разделять лекции и семинары.

На рис. 2 показано решение задачи “в лоб”.

![Image alt](https://github.com/IlnarAhm/php-design-patterns/raw/main/Strategy/img/UML-2.png)

<figcaption align="center"><b>Puc. 2.</b> <i>Неудачная структура наследования</i></figcaption>
<br/><br/>

На рис. 2 приведена явно неудачная иерархия. Такое дерево наследования нельзя в дальнейшем использовать для того, чтобы управлять механизмами оплаты, не дублируя большие блоки функциональных средств. Эти механизмы оплаты повторяются в семействах классов Lecture и Seminar.

### Решение проблемы

Для решения данной задачи можно воспользоваться шаблоном Strategy, который служит для перемещения ряда алгоритмов в отдельный тип данных.

Мы создали еще один абстрактный класс, _CostStrategy_, в котором определены абстрактные методы _cost()_ и _chargeType()_. Методу _cost()_ необходимо передать экземпляр класса _Lesson_, который он будет использовать для расчета стоимости занятия. Мы предоставляем две реализации класса CostStrategy. Объекты типа Lesson оперируют только типом CostStrategy, а не конкретной реализацией, поэтому можно в любое время добавить новые алгоритмы расчета стоимости, создавая подклассы на основе класса CostStrategy. И для этого не придется вносить вообще никаких изменений в классы Lesson.

![Image alt](https://github.com/IlnarAhm/php-design-patterns/raw/main/Strategy/img/UML-3.png)

<figcaption align="center"><b>Puc. 3.</b> <i>Перемещение алгоритмов в отдельный тип данных</i></figcaption>
<br/><br/>

Приведем упрощенную версию нового класса Lesson, показанного на рис. 3:

```php
abstract class Lesson
{
    public function __construct(
        private int $duration,
        private CostStrategy $costStrategy
    ) {
    }

    public function cost(): int
    {
        return $this->costStrategy->cost($this);
    }

    public function chargeType(): string
    {
        return $this->costStrategy->chargeType();
    }

    public function getDuration()
    {
        return $this->duration;
    }
}

class Lecture extends Lesson
{
    // Реализации, специфичные для класса Lecture...
}

class Seminar extends Lesson
{
    // Реализации, специфичные для класса Seminar...
}
```

-   [Lesson](https://github.com/IlnarAhm/php-design-patterns/tree/main/Strategy/Lesson.php)
-   [Lecture](https://github.com/IlnarAhm/php-design-patterns/tree/main/Strategy/Lecture.php)
-   [Seminar](https://github.com/IlnarAhm/php-design-patterns/tree/main/Strategy/Seminar.php)

Конструктору класса _Lesson_ передается объект типа _CostStrategy_, который он сохраняет в виде свойства. В методе _Lesson::cost()_ просто делается вызов _CostStrategy::cost()_. Аналогично в методе _Lesson::chargeType()_ делается вызов _CostStrategy::chargeType()_. Такой явный вызов метода из другого объекта для выполнения запроса называется делегированием. В рассматриваемом здесь примере объект типа _CostStrategy_ является делегатом класса _Lesson_. А класс _Lesson_ снимает с себя ответственность за расчет стоимости занятия и возлагает эту обязанность на реализацию класса _CostStrategy_. Ниже показано, каким образом осуществляется делегирование:

```php
public function cost(): int
{
    return $this->costStrategy->cost($this);
}
```

А вот определение класса _CostStrategy_ вместе с реализующими его дочерними классами:

```php
abstract class CostStrategy
{
    abstract public function cost(Lesson $lesson): int;
    abstract public function chargeType(): string;
}

class TimedCostStrategy extends CostStrategy
{
    public function cost(Lesson $lesson): int
    {
        return ($lesson->getDuration() * 5);
    }
    public function chargeType(): string
    {
        return "Почасовая оплата";
    }
}

class FixedCostStrategy extends CostStrategy
{
    public function cost(Lesson $lesson): int
    {
        return 30;
    }
    public function chargeType(): string
    {
        return "Фиксированная ставка";
    }
}
```

-   [CostStrategy](https://github.com/IlnarAhm/php-design-patterns/tree/main/Strategy/CostStrategy.php)
-   [TimedCostStrategy](https://github.com/IlnarAhm/php-design-patterns/tree/main/Strategy/TimedCostStrategy.php)
-   [FixedCostStrategy](https://github.com/IlnarAhm/php-design-patterns/tree/main/Strategy/FixedCostStrategy.php)

Теперь во время выполнения программы можно легко изменить способ расчета стоимости занятий, выполняемый любым объектом типа _Lesson_, передав ему другой объект типа _CostStrategy_. Такой подход способствует созданию довольно гибкого кода. Вместо того чтобы статично встраивать функциональность в структуры кода, можно комбинировать объекты и менять их сочетания динамически:

```php
$lessons[] = new Seminar(4, new TimedCostStrategy());
$lessons[] = new Lecture(4, new FixedCostStrategy());

foreach ($lessons as $lesson) {
    print "Оплата за занятие {$lesson->cost()}. ";
    print " Тип оплаты: {$lesson->chargeType()}\n";
}
```

Выполнение данного фрагмента кода приведет к следующему результату:

```zsh
Оплата за занятие 20. Тип оплаты: Почасовая оплата
Оплата за занятие 30. Тип оплаты: Фиксированная ставка
```

Как видим, одно из следствий принятия такой структуры состоит в том, что мы распределили обязанности классов. Объекты типа _CostStrategy_ ответственны только за расчет стоимости занятия, а объекты типа _Lesson_ управляют данными занятия.

Итак, композиция позволяет сделать код намного более гибким, поскольку можно комбинировать объекты и решать задачи динамически намного большим количеством способов, чем при использовании одной лишь иерархии наследования. Но при этом исходный код может стать неудобочитаемым. В результате композиции, как правило, создается больше типов данных с отношениями, которые не настолько предсказуемы, как отношения наследования. Поэтому понять отношения в такой системе немного труднее.
