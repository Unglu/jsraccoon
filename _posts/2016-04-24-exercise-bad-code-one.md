---
title:  "Плохой код 1"
categories: JavaScript
date:   2016-03-21 15:00:00 +0300
author_name: "Евгений Бовыкин"
author: missingdays
type: exercise
identifier: exercise-bad-code-1

description: "Исправляем плохой код"

tags: [javascript]
---

> Этот цикл статей вдохновлен книгами Роберта Мартина "Чистый код" и Брайна Кернигана и Роба Пайка "Практика программирования". В нем я постараюсь привести реальные примеры плохого кода и возможные пути его исправления с подробными пояснениями.

Эта статья начинает серию "Плохой код", в котором вам предлагается исправить различные ошибки и недостатки небольшого куска кода. Ошибки могут быть разные - код может быть неправильным, код может быть неправильно спроектирован, код может "плохо пахнуть" и так далее.

Важно понимать, что не существует единственно правильного решения. Но при каждом исправлении важно понимать, почему вы это исправление делаете.

Для разминки предлагаю исправить такой код. Он специально утрирован, чтобы показать, что даже в нескольких строках кода может быть большое количество проблем. 

{% highlight javascript %}

class Character {
    handleInput(input){
        if(input.keyCode == 87) { // 'w'
            this.y += 5;
        } else if(input.keyCode == 68) { // 'd'
            this.x += 5;
        } else if(input.keyCode == 83) { // 's'
            this.y -= 5;
        } else if(input.keyCode == 65) { // 'a'
            this.x -= 5;
        }
    }
    ...

{% endhighlight %}

Для ясности: `handleInput` - метод какого-то класса, возможно персонажа, которым мы можем управлять.

## Исправления

### Явные недочеты

Начнем с очевидных моментов.

Каждый раз, когда вы видете нетривиальную строку и рядом с ней комментарий, который ее поясняет - это плохая строка. `if(input.keyCode == 87) { // 'w'` - неужели каждый раз придется писать рядом `// 'w'`? А если мы поменяем ее на `if(input.keyCode == 86) ... ` нам придется менять и сам комментарий. Один раз мы это сделаем, другой раз забудем. И получится врущий комментарий `if(input.keyCode == 86) // 'w'`. Это одна из причин, почему таких комментариев стоит избегать.

Кстати, об изменениях. Что, если ниже у нас есть еще пара методов, в которых тоже есть злополучные `if(input.keyCode == 87)`, а мы вдруг захотим вместо `WASD` использовать стрелки клавиатуры? Придется в каждом месте руками изменять 87 на 35, 68 на 39 и так далее. Не хорошо.

#### Первое изменение

Попробуем вынести так называемые "магические константы" в отдельный объект.

{% highlight javascript %}


// Запишем только нужные нам коды
let KEY_CODE = {
    A: 65,
    D: 68,
    S: 83
    W: 87,
};

class Character {
    handleInput(input){
        if(input.keyCode == KEY_CODE.W) {
            this.y += 5;
        } else if(input.keyCode == KEY_CODE.D) {
            this.x += 5;
        } else if(input.keyCode == KEY_CODE.S) {
            this.y -= 5;
        } else if(input.keyCode == KEY_CODE.A) {
            this.x -= 5;
        }
    }
    ...

{% endhighlight %}

Как видите, мы избавились от ненужных комментариев, при этом читаться код хуже не стал.

#### Второе изменение

Однако, если мы захотим изменить клавиши управления нашим персонажем, у нас получится что-то вроде 

{% highlight javascript %}

let KEY_CODE = {
    A: 37,
    D: 39,
    S: 40,
    W: 38,
};

{% endhighlight %}

Опять получается, что наш код врет. Мы говорим, что букве А соответсвует число 37, хотя это не так. Конечно, наш код будет работать, и наш персонаж будет двигаться при помощи стрелок, однако в дальнейшем такой код может легко запутать всех, кто будет над ним работать, включая вас, самого автора.

Для исправления этой проблемы достаточно изменить имена переменных

{% highlight javascript %}


let INPUT_KEY = {
    LEFT: 65
    RIGHT: 68,
    DOWN: 83
    UP: 87,
};

class Character {
    handleInput(input){
        if(input.keyCode == INPUT_KEY.UP) {
            this.y += 5;
        } else if(input.keyCode == INPUT_KEY.RIGHT) {
            this.x += 5;
        } else if(input.keyCode == INPUT_KEY.DOWN) {
            this.y -= 5;
        } else if(input.keyCode == INPUT_KEY.LEFT) {
            this.x -= 5;
        }
    }
    ...

{% endhighlight %}

Теперь из имен констант нам сразу становится смысл их предназначения, к тому же мы можем легко их переопределять, не боясь, что они потеряют смысл.

#### Третье изменение

Теперь подумаем, что, если мы захотим изменить скорость передвижения нашего персонажа. Нам придется изменять руками число 5 на значение, которое мы хотим, и делать это по всему коду. Нехорошо, учитывая, что число 5 в другом месте может означать совершенно другое значение, и, если у нас много таких магических констант, каждое такое исправление превратиться в ад. И если такое происходит - ваш код точно плох.

Определим методы для движения нашего персонажа, а также вынесем скорость в отдельные переменные, которые легко изменять.

{% highlight javascript %}


class Character {

    constructor(){
        ...

        this.speed = 5;
    }

    handleInput(input){
        if(input.keyCode == INPUT_KEY.UP) {
            this.moveUp();
        } else if(input.keyCode == INPUT_KEY.RIGHT) {
            this.moveRight();
        } else if(input.keyCode == INPUT_KEY.DOWN) {
            this.moveDown();
        } else if(input.keyCode == INPUT_KEY.LEFT) {
            this.moveLeft();
        }
    }

    moveUp(){
        this.y += this.speed;
    }

    moveDown(){
        this.y -= this.speed;
    }

    moveRight(){
        this.x += this.speed;
    }

    moveLeft(){
        this.x -= this.speed;
    }
    ...

{% endhighlight %}

Мы получили еще один плюс - теперь метод `handleInput` читается гораздо легче. Попробуйте произнести его вслух - "если нажата клавиша вверх - двигайся вверх, иначе, если нажата клавиша вправо - двигайся враво и так далее". Намного лучше, чем "если код клавишы 87, увеличь свою координату на 5, иначе..." - сразу становятся понятны намерения разработчка, сразу ясно, что этот код вообще делает.


### Архитектура

#### Четвертое изменение

Теперь самый главный вопрос, который у вас должен возникнуть - почему сам персонаж должен отвечать за обработку нажатия клавиш, и почему сами константы `INPUT_KEY` находятся рядом с ним? Нам явно нужен отдельный модуль, который будет отвечать за взаимодействие с пользователем, а наш персонаж лишь будет получать оповещения о тех или иных событиях. Реализаций такого поведения может быть много в зависимости от ваших предпочтений и более тонких нужд, я предлагаю использовать стандартную событийную модель подписок. Наш персонаж будет подписываться на события ввода и выполнять соответсвующее действие.

Ниже приведен лишь общий код, а не полная и конкретная реализация. `InputConroller` по сути является стандартным `Observable`. Про него вы можете почитать и реализовать самостоятельно в [этой статье](http://jsraccoon.ru/exercise-observable).

{% highlight javascript %}

// Модуль InputConroller

let INPUT_KEYS = {
    LEFT: 65
    RIGHT: 68,
    DOWN: 83
    UP: 87,
};

class InputConroller {
    
    onInput(input, callback){
        ...
    }

// Модуль Character

class Character {
    consructor(){
        ...
        SubscribeForInputs();
    }

    SubscribeForInputs(){
        inputConroller.onInput('UP', () => this.moveUp());
        inputConroller.onInput('RIGHT', () => this.moveRight());
        inputConroller.onInput('DOWN', () => this.moveDown());
        inputConroller.onInput('LEFT', () => this.moveLeft());
    }
    ...
{% endhighlight %}

Теперь не только наш персонаж, но вообще все объекты в игре могут легко подписываться на события ввода. Более того, в сам `InputContoller` может быть внесена логика смены управления, при этом весь остальной код трогать не придется - мы ведь просто подписываемся на событие 'UP', а какая клавиша за него отвечает - дело самого обработчика.

## Вывод

Вот так в 10 строчках кода нашлось большое количество проблем, для решения которых потребовалось много шагов. Надеюсь, этой статьей мне удалось зародить в вас интерес к такой важной теме, как качественное написание кода. 

Дальше код будет не только хуже, он будет коварнее. Ошибки будут таиться в самых неожиданных местах.

Спасибо!

## P.S.
Еще одна важная проблема, которую я хотел затронуть - неправильное движение персонажа. Однако эта тема далека непосредственно он качества кода, здесь речь идет об игровой механике, и материала здесь на отдельную статью :) А если вкратце - попробуйте посчитать скорость нашего персонажа вдоль осей Х и Y и вдоль диагоналей, если нажимать одновременно W+A например.