# Лекция 7. Прототипы

[Назад](../../README.md)

## Начало

> **Объект** - это коллекция свойств, а свойство - это ассоциация между именем (или ключём) и значением. Самый популярный способ создания объекта - это фигурные скобки, с добавлением свойств между ними.

В JavaScript объект - это отдельная сущность со свойствами и типом. Его можно сравнить с животным. Животное - это объект со свойствами, которое имеет цвет, вид, вес и т.д. 

```jsx
let animal = {}
animal.name = 'Niko'
animal.energy = 10

animal.eat = function(amount) {
	console.log(`${this.name} is eating.`)
	this.energy += amount
}

animal.sleep = function(length) {
	console.log(`${this.name} is sleeping.`
	this.energy += length
}

animal.play = function(length) {
	console.log(`${this.name} is playing.`)
	this.energy -= length
}
```

*Пример создания объекта*

В этом коде создаётся пустой объект `animal`, к которому мы добавили свойства с именем и энергией.

А так же свойства-функции, которые позволяют животному есть, играть и спать, что меняет текущее количество энергии.

## Функциональная инсталляция

Если мы хотим создать несколько животных, а не одно, мы можем создать для этого соответсвующую функцию. Нам нужно для этого обернуть весь предыдущий код в функцию, которую мы будем вызывать для создания объекта. Эта функция будет возвращать сам объект, который мы создаём. Этот шаблон называется **функциональной инсталляцией**, а сама функция - **функция конструктора**, т.к. она отвечает за построение объекта.

```jsx
function Animal (name, energy) {
  let animal = {}
  animal.name = name
  animal.energy = energy

  animal.eat = function (amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }

  animal.sleep = function (length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }

  animal.play = function (length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }

  return animal
}

const niko = Animal('Niko', 7)
const miko = Animal('Miko', 10)
```

*Пример создания функции конструктора*

Теперь нам достаточно вызывать эту функцию, чтобы получать новые "особи" этих объектов. Но в таком подходе есть минус: хоть методы и являются идентичными у всех "особей", они дублируются при каждом создании новой "особи" и занимают лишнюю память.

Мы можем решить это следующим образом: мы можем собрать все методы в отдельный объект и при создании "особей" нашего животного, просто передавать каждой особи ссылки на соответствующие методы. Такой шаблон называется **функциональная инсталляция с общими методами**.

```jsx
const animalMethods = {
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  },
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  },
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

function Animal (name, energy) {
  let animal = {}
  animal.name = name
  animal.energy = energy
  animal.eat = animalMethods.eat
  animal.sleep = animalMethods.sleep
  animal.play = animalMethods.play

  return animal
}

const niko = Animal('Niko', 7)
const miko = Animal('Miko', 10)
```

## Object.create

`Object.create` позволяет создать объект на основе другого, и каждый раз, когда на этом объекте не удаётся выполнить поиск свойств, он может проконсультироваться с другим объектом, чтобы проверить, имеет ли этот *другой* объект нужное свойство.

```jsx
const animal = {
	name: 'Niko',
	energy: 10,
	breed: 'Norwegian Forest'
}

const child = Object.create(animal)
child.name = 'Miko'
child.energy = 7

console.log(child.name) // Miko
console.log(child.energy) // 7
console.log(child.breed) // Norwegian forest
```

*Пример создания объекта при помощи `Object.create`*

Таким образом, если в `child` нет какого-то необходимого нам свойства (в данном случае `breed`), программа автоматически будет искать это свойство в `animal`.

Зная это, можно упростить предыдущим пример:

```jsx
const animalMethods = {
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  },
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  },
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

function Animal (name, energy) {
	// создаём объект, который привязан к `animalMethods`
	let animal = Object.create(animalMethods)
  animal.name = name
  animal.energy = energy

  return animal
}

const miko = Animal('Miko', 7)
const niko = Animal('Niko', 10)

miko.eat(10)
niko.play(5)
```

*Пример использования `Object.create`*

Мы можем ещё улучшить данный код, т.к. в данный момент наши методы лежат в отдельном объекте, а мы можем поместить их в прототип функции конструктора.

`prototype` это внутреннее свойство, которе присутствует у всех объектов в JavaScript и является ссылкой на объект-родителя.

```jsx
function Animal (name, energy) {
  let animal = Object.create(Animal.prototype)
  animal.name = name
  animal.energy = energy

  return animal
}

Animal.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

Animal.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}

Animal.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}

const miko = Animal('Miko', 7)
const niko = Animal('Niko', 10)

miko.eat(10)
niko.play(5)
```

*Пример с использованием прототипа*

## Как выглядит прототип?

Мы можем увидеть структуру прототипов воспользовавшись консолью. Для начала создаём объект:

```jsx
const animal = {
	name: 'Niko',
	energy: 10,
	breed: 'Norwegian Forest'
}
```

После этого нам достаточно вызвать объект и посмотреть его развёрнутую структуру:

```jsx
> animal
<	{name: "Niko", energy: 10, breed: "Norwegian forest"}
	breed: "Norwegian forest"
	energy: 10
	name: "Niko"
		__proto__:
			constructor: ƒ Object()
			hasOwnProperty: ƒ hasOwnProperty()
			isPrototypeOf: ƒ isPrototypeOf()
			propertyIsEnumerable: ƒ propertyIsEnumerable()
			toLocaleString: ƒ toLocaleString()
			toString: ƒ toString()
			valueOf: ƒ valueOf()
			__defineGetter__: ƒ __defineGetter__()
			__defineSetter__: ƒ __defineSetter__()
			__lookupGetter__: ƒ __lookupGetter__()
			__lookupSetter__: ƒ __lookupSetter__()
			get __proto__: ƒ __proto__()
			set __proto__: ƒ __proto__()
```

*Пример структуры прототипа*

У всех прототипов есть общие свойства: `constructor` и `__proto__`. Свойство `constructor` указывает на функцию-конструктор, с помощью которой этот объект создавался, а свойство `__proto__` указывает на следующий прототип в цепи (или `null`, если это последний прототип).

Свойство `__proto__` является *геттером* и *сеттером* для внутренного свойство `[[Prototype]]` и находится в `Object.prototype`. Из-за этого:

```jsx
const animal = {
	name: 'Niko',
	energy: 10,
	breed: 'Norwegian Forest'
}
animal.toString() // '[object Object]'
// метод toString() берётся из Object.prototype
animal.__proto__ = null // устанавливаем прототип объекта null
animal.toString() // TypeError
// такого метода не найдено, т.к. нет прототипа
animal.__proto__ = Object.prototype // пытаемся восстановить связь
animal.toString() // TypeError
// прототип не вернулся
```

*Доказательство того, что `__proto__` является унаследованным свойством от `Object.prototype`, и не присутствует в самом объекте*

Как только мы удалили прототип из объекта, `__proto__` стало недоступным и его уже никак не вернуть обратно.

Рассмотрим другой пример:

```jsx
const animal = {
  name: 'Niko',
  energy: 10,
  breed: 'Norwegian forest'
}
const child = {
  name: 'Miko',
  energy: 7,
}

child.__proto__ = animal;
```

*Записываем `animal` к `child` в `__proto__`*

В консоли Chrome `child` будет выглядит так:

```jsx
> child
<	{name: "Miko", energy: 7}
	energy: 7
	name: "Miko"
		__proto__:
		breed: "Norwegian forest"
		energy: 10
		name: "Niko"
			__proto__:
			constructor: ƒ Object()
			hasOwnProperty: ƒ hasOwnProperty()
			isPrototypeOf: ƒ isPrototypeOf()
			propertyIsEnumerable: ƒ propertyIsEnumerable()
			toLocaleString: ƒ toLocaleString()
			toString: ƒ toString()
			valueOf: ƒ valueOf()
			__defineGetter__: ƒ __defineGetter__()
			__defineSetter__: ƒ __defineSetter__()
			__lookupGetter__: ƒ __lookupGetter__()
			__lookupSetter__: ƒ __lookupSetter__()
			get __proto__: ƒ __proto__()
			set __proto__: ƒ __proto__()
```

*Вид объекта `child` в консоли Chrome*

А теперь уберём связь между `animal` и `Object.prototype`:

```jsx
animal.__proto__ = null
```

Теперь, при попытке взглянуть на `child` в консоли видим:

```jsx
> child
< {name: "Miko", energy: 7}
		energy: 7
		name: "Miko"
			__proto__:
				breed: "Norwegian forest"
				energy: 10
				name: "Niko"
```

Связь с `Object.prototype` у `animal` разорвана и `__proto__` возвращает `undefined` даже у дочернего объекта `child`. 

Использование `__proto__` не приветствуется и рекомендуется использование методов `Object.getPrototypeOf` и `Object.setPrototypeOf`.

## Функции и конструкторы

```jsx
function Animal(name, energy) {
	this.name = name;
	this.energy = energy;
}

const animal = new Animal('Miko', 10);
```

*Описываем функцию конструктор и создаём с её помощью объект, но с использованием оператора `new`*

Теперь можем взглянуть на `prototype` функции `Animal`:

```jsx
> Animal.prototype
< {constructor: ƒ}
		// Animal.prototype
		constructor: ƒ Animal(name, energy)
		arguments: null
		caller: null
		length: 2
		name: "Animal"
		prototype: {constructor: ƒ}
		__proto__: ƒ ()
		[[FunctionLocation]]: VM44:1
		[[Scopes]]: Scopes[2]
  // Object.prototype
	__proto__: Object
```

При объявлении функции, у неё автоматически создаётся свойство `prototype` чтобы её можно было использовать как конструктор. Таким образом свойство `prototype` функции не имеет отношения к прототипу самой функции, а задаёт прототипы для дочерних объектов. Это позволит реализовать наследование и добавлять нвоые методы, например так:

```jsx
Animal.prototype.greeting = function () {
	return `Hello! My name is ${this.name}.`
}
```

Теперь прототип выглядит следующим образом:

```jsx
> Animal.prototype
< {greeting: ƒ, constructor: ƒ}
		greeting: ƒ ()
		constructor: ƒ Animal(name, energy)
		__proto__: Object
```

И вызов `animal.greeting()` вернёт `Hello! My name is Miko.`.

## Что такое `new`?

Оператор `new` позволяет разработчикам создать экземпляр указанного пользователем типа объекта, или одного из встроенных типов объектов, которые содержат функцию-конструктор.

Сам `new` не несёт никаких скрытых действий и всё что он делает можно повторить силами самого языка, таким образом можно написать свой `new`:

```jsx
function constructor_new(constructor, args) {
	const self = Object.create({});
	const constructorValue = constructor.apply(self, args) || self;
	function isPrimitive(val) {
		return val !== Object(val);
	}
	return isPrimitive(constructorValue) ? self : constructorValue;
}
constructor_new(Animal, ['Miko', 10])
```

*Собственная реализация оператора `new`*

При вызове `new` выполняет следующие действия:

1. Создаёт новый объект `self`.
2. Записывает свойство `prototype` функции конструктора в прототип объекта `self`.
3. Вызывает функцию конструктор с объектом `self` как аргументом `this`.
4. Возвращает `self` если конструктор вернул примитивное значение, в обратном случае возвращает значение с конструктора.

## Наследование и цепочка прототипов

Предположим что мы хотели бы раздать индивидуальные свойства для конкретных животных. Для котов, например, мы бы дали имя, уровень энергии и способность есть, спать и играть. Уникальными для котов были бы свойства для породы и возможности мяукать. В ES5 наш класс `Cat` может выглядеть так:

```jsx
function Cat (name, energy, breed) {
	this.name = name
	this.energy = energy
	this.breed = breed	
}

Cat.prototype.eat = function (amount) {
	console.log(`${this.name} is eating.`)
	this.energy += amount;
}

Cat.prototype.sleep = function (length) {
	console.log(`${this.name} is sleeping.`)
	this.energy += length;
}

Cat.prototype.play = function (length) {
	console.log(`${this.name} is playing.`)
	this.energy -= length
}

Cat.prototype.meow = function () {
	console.log('Moew-Meow!')
	this.energy -= .1
}

const miko = new Cat('Miko', 10, 'Norwegian Forest')
```

Мы создали класс, который похож на `Animal` из предыдущих примеров. Если бы нам понадобился бы класс `Dog`, то мы могли бы снова продублировать всё описание, только добавить какие-то свойства, которые уже были бы уникальными для собаки, и так для всех животных. Чтобы не писать общие свойства каждый раз по новой, это можно оптимизировать.

```jsx
function Animal (name, energy) {
	this.name = name
	this.energy = energy
}

Animal.prototype.eat = function (amount) {
	console.log(`${this.name} is eating.`)
	this.energy += amount
}

Animal.prototype.sleep = function (length) {
	console.log(`${this.name} is sleeping.`)
	this.energy += length
}

Animal.prototype.play = function (length) {
	console.log(`${this.name} is playing.`)
	this.energy -= length
}
```

*Описание класса `Animal`*

Создание котов выглядит так:

```jsx
function Cat (name, energy, breed) {}
```

Таким образом для создания кота нам нужно вызвать функцию `Cat` с оператором `new`. В эту функцию мы передадим имя, количество изначальной энергии особи и её породу. В особях класса `Animal` уже заложено имя и рабоа с энергией, по этому мы можем использовать `Animal` как основу для наших котов, просто потом добавляя только уникальные свойства, присущие *только* котам. Для этого мы можем использовать метод `call`.

```jsx
// ... описание класса Animal

function Cat (name, energy, breed) {
	Animal.call(this, name, energy)
	this.breed = breed
}

const miko = new Cat('Miko', 10, 'Norwegian forest')

miko.name // Miko
miko.energy // 10
miko.breed // Norwegian forest
```

*Пример наследования класса*

Теперь благодаря строке `Animal.call(this, name, energy)` каждый котик будет иметь имя и уровень энергии, хотя в самом описании объекта котика этих полей нет.

Но пока это работает только со свойствами. В данный момент котик не наследует методы, которые хранятся в `Animal.prototype`, которые позволили бы ему есть, играть и спать.

Мы можем связать их используя `Object.create()`:

```jsx
function Cat(name, energy, breed) {
	Animal.call(this, name, energy)
	this.breed = breed
}
Cat.prototype = Object.create(Animal.prototype)
```

Как именно это помогло рассмотрим дальше.

Полный код:

```jsx
function Animal (name, energy) {
  this.name = name
  this.energy = energy
}

Animal.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}

Animal.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}

Animal.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}

function Cat (name, energy, breed) {
  Animal.call(this, name, energy)

  this.breed = breed
}

Cat.prototype = Object.create(Animal.prototype);
```

Теперь мы можем создать кота и посмотреть на его функционал:

```jsx
const miko = new Cat('Miko', 10, 'Norwegian forest')

miko.name // Miko
miko.energy // 10
miko.breed // Norwegian forest
```

Вроде бы всё то же самое однако теперь мы можем использовать методы из `Animal.prototype`. Это происходит следующим образом:

1. Вызов, например, `miko.eat(10)`
2. Есть ли у объекта `miko` метод `eat`? Нет.
3. Есть ли у объекта `Cat.prototype` метод `eat`? Нет.
4. Есть ли у объекта `Animal.prototype` метод `eat`? Есть! Вызываем его.

JavaScript знает что после 2го пункта стоит посмотреть в `Cat.prototype` по следующей причине:

Когда мы вызываем функцию-конструктор с оператором `new` он неявно добавляет в её код некоторые строки:

```jsx
function Cat (name, energy, breed) {
	// this = Object.create(Cat.prototype)
	Animal.call(this, name, energy)
	
	this.breed = breed
	// return this
} 
```

*Закоментированные строки добавляеят оператор `new`*

Таким образом `new` связывает наш новый объект с `Cat.prototype`.

Но при такой реализации кода есть проблема:

У каждого экземпляра можно получить его функцию-конструктор при помощи `[имя объекта].constructor`. То есть например:

```jsx
function Animal (name, energy) {
	this.name = name
	this.energy = energy
}

const niko = new Animal('Niko', 7)
console.log(niko.constructor) // [Function: Animal]
```

Самое свойство `constructor` находится в `Animal.prototype`. Т.к. в `niko` этого свойства нет, поиск делегируется в `Animal.prototype`. 

В нашей реализации наследования мы перезаписываем `Cat.prototype` объектом, который делегирует `Animal.prototype`.

Это означает что любые экземпляры `Cat`, которые подписываются на `instance.constructor` будут получать конструктор `Animal`, а не конструктор `Cat`. 

Происходит это так:

1. Есть ли у `niko` свойство `constructor`? Нет.
2. Есть ли у `Cat.prototype` свойство `constructor`? Нет, мы его удалили, когда перезаписали `Cat.prototype`.
3. Есть ли у `Animal.prototype` свойство `constructor`? Да, возвращаем его значение. (`[Function: Animal]`)

Это легко исправить, достаточно просто добавить правильное свойство `constructor` в `Cat.prototype`, после того как перезапишем его:

```jsx
function Cat (name, energy, breed) {
	Animal.call(this, name, energy)

	this.breed = breed
}

Cat.prototype = Object.create(Animal.prototype)

Cat.prototype.meow = function () {
  console.log('Meow-Meow!')
  this.energy -= .1
}

// добавляем правильный конструктор
Cat.prototype.constructor = Сat
```

## Дополнительные материалы

[getify/You-Dont-Know-JS](https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch5.md)

[What Makes JavaScript JavaScript? Prototypal Inheritance](https://dmitripavlutin.com/javascript-prototypal-inheritance/)

[JavaScript properties: inheritance and enumerability](https://2ality.com/2011/07/js-properties.html)

[JavaScript - Prototype](https://codeburst.io/javascript-prototype-cb29d82b8809)

## Задание

[Croftyland/YDKJS-kottans](https://github.com/Croftyland/YDKJS-kottans/tree/master/3.%20this%26prototypes)
