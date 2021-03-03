# Лекция 6. Objects & this

**Лектор**: Анастасия Машошина
**Модуль**: YDKJS

[Назад](../../README.md)

# Объекты
> Объект - это коллекция свойств вида *ключ*-*значение*.

## Любой ключ - строка/символ

В объекте значениями могут быть любые типы данных, а вот ключами - только **сроки** и **символы**.

Пример:
```js
const myObj = {};
myObj[true] = "yes";
Object.keys(myObj).forEach(key => {
  console.log(typeof key)
}) // string
```

Другой пример:
```js
const myObj = {};
const anotherObj = {};
myObj[anotherObj] = "test";
console.log(myObj); // {[object Object]: "test"}
```

JS автоматически постарается привести название ключа к строке.

То же самое действует и на массивы, т.к. массивы - это частные случаи объектов.

Пример:
```js
const myArr = [1, 2, 3]
Object.keys(myArr).forEach(key => console.log(typeof key)) // string string string
```

## Дескрипторы свойств
На первый взгляд кажется, что объект просто состоит из пар ключ-значение. Хотя на самом деле всё немного сложнее:

Пример:
```js
const myObj = {key: 'value'}
Object.getOwnPropertyDescriptor(myObj, 'key')
```
Результатом последней строки будет объект-дрескриптор свойства, которое мы ему передали. Выглядит так:
```js
{
  configurable: true,
  enumerable: true,
  value: "value",
  writable: true
}
```

В этом объекте есть следующие свойства:
  - `configurable` - можем ли мы изменять остальные свойства в дескрипторе
  - `enumerable` - будет ли это свойство включено в перебор, например, когда мы используем `Object.keys()`
  - `value` - значение, которое записано в свойстве
  - `writable` - можем ли мы перезаписать значение свойства

**Альтернативное создание свойства у объекта**:
Мы можем добавлять объекту свойства при помощи `Object.defineProperty(targetObj, propertyName, propertyDescriptor)`.

Чтобы увидеть стандартные значения мы можем создать новое свойство с пустым дескриптором:

```js
const myObj = {};
Object.defineProperty(myObj, 'newProp', {})
Object.getOwnPropertyDescriptor(myObj, 'newProp')
```

Результатом последней строки будет:
```js
{
  configurable: false,
  enumerable: false, 
  value: undefined,
  writable: false
}
```

## Data properties vs accessor properties
Пример:
```js
const creature = {
  name: 'Jack',
  get greeting() {
    return `Hello, I am %{this.name}`
  }
}
```
Свойства данных - это просто значения, как в данном случае - `name`.

А свойства доступа - это особые функции, которые позволяют нам выполнять некоторые действия каждый раз когда кто-то хочет записать или считать свойство с объекта.

В данном примере, если мы напишем: `creature.greeting`, мы получим в ответ `Hello, I am Jack`. Хотя функцию мы не вызывали, если мы пытаемся достать из свойства значение, то будет вызвана `get`-функция с соответствующим названием.

Дескриптор свойства для `greeting` в данном случае будет выглядеть так:
```js
{
  configurable: true,
  enumerable: true,
  get: function greeting()
  set: undefined
}
```
(*в get хранится ссылка на функцию*)

## Object.freeze() & Object.seal()
Эти методы ограничивают расширения объектов. 

- `Object.seal()` не позволяет добавлять новые свойства в объект
- `Object.freeze()` не позволяет меня объект вообще

Пример:
```js
const newObj = {
  key: 'value'
}
newObj.otherProps = 'hello'
Object.seal(newObj)
newObj.yetAnotherProp = 'world'
```
В данном случае свойство `yetAnotherProp` не будет записано в объект. Если бы мы вызвали последнюю строку при включённом "строгом режиме", то получили бы ошибку `TypeError`.

При этом мы всё ещё можем перезаписывать значения, которые были добавлены до "запечатывания" объекта. Для свойств внутри объекта, который "запечатали" в дескрипторе свойства свойство `configurable` установлено в `false`.

Мы можем ужесточить ограничения для объекта, "заморозив" его при помощи `Object.freeze(newObj)`. В таком случае мы не сможем даже изменять те свойства, которые в него уже записаны, т.к. в их дескрипторах свойство `writable` будет установлено как `false`.

## Computed properties
Мы так же можем создавать объекты со свойствами, названия которых будут хранится в переменных или как-то вычисляться.

Пример:
```js
const key = 'test'
const yetAnotherObj = {
  [key]: 'hello'
}
console.log(yetAnotherObj) // {test: 'hello'}
```

## in & for...in
Оператор `in` проверяет есть ли какое-то свойство в объекте и возвращает нам `true`/`false`.

Пример:
```js
const testObj = {
  a: 2
}
console.log('a' in testObj) // true
```

Оператор `for...in` позволяет создать цикл, перебирая ключи объекта. 

Пример:
```js
const testObj = {
  a: 2,
  b: 3
}
for(const key in testObj) {
  console.log(key)
}
// a
// b
```

# this
`this` - это контекст выполнения функции, он определяется в момент её вызова, зависит исключительно от того как она будет вызвана.

Пример:
```js
function sayName() {
  console.log(this)
  console.log(this.fullName)
}
sayName()
// Window {...}
// undefined
```

Если мы просто так вызовем функцию в пустоте - `this` будет указан на `default binding`. Если мы вызываем функцию не в "строгом режиме", то `this` будет указывать на объект `Window`.

Если мы запустим функцию в пустоте но в "строгом режиме", `this` будет установлено значение `undefined`.

## Изменение контекста
Представим, что у нас есть объект:
```js
const kottanObject = {
  city: 'Kyiv',
  language: 'JS'
}
```

У нас так же есть функция:
```js
function greet() {
  console.log(`Hello! I'm from ${this.city}. I write in ${this.language}.`)
}
```

Мы хотим вызывать нашу функцию в контекста нашего объекта. 

### Добавить функцию как свойство
Мы можем просто добавить в наш объект ссылку на нашу функцию.

```js
kottanObject.greet = greet
kottanObject.greet()
// Hello! I'm from Kyiv. I write in JS.
```
Такой метод вызова (через точку) называется **Implicit binding**.

### Передача метода как коллбек
Так как контекст функции определяется в момент её вызова, иногда мы можем потерять этот контекст, когда куда-то передаём функцию. Например:

```js
setTimeout(kottanObject.greet, 0)
```

В данном примере мы не вызываем функцию сразу, по этому implicit binding не работает. В данном случае мы просто указываем что в коллбек нужно передать метод `greet` объекта `kottanObject`. Мы передаём само тело функции, т.к. вызова нет - нет никакой привязки контекста. Вызываться она будет потом самим браузером без контекста, по этому будет выведено `Hello! I'm from undefined. I write in undefined`.

### Explicit binding
Для явного привязывания контекста у каждой функции в прототипе есть некоторые удобные методы:
  - `call()`:
    Позволяет вызывать функцию, но указать ей контекст. Синтаксис:
      ```js
        functionName.call(objectName, arg1, arg2, ...args)
      ```
  - `apply()`:
    Позволяет делать то же самое, что и `call()`, но аргументы передаются массивом:
      ```js
        functionName.call(objectName, [arg1, arg2, ...args])
      ```
  - `bind()`:
    Позволяет получить функцию с привязанным контекстом. Не производит вызов функции, а просто возвращает новую функцию с привязанным контекстом.
      ```js
        const newFunc = functionName.bind(objectName)
      ```

### При использовании классов
Пример:
```js
const Student = function() {
  console.log(this)
}
```
Если просто написать в консоли `Student()`, то в таком случае в консоль выведется объект `Window`, т.к. `this` ни к чему не привязан. Но, если создать новый объект этого класса, то есть написать `new Student`, то в консоль будет выведен объект `Student {}`.
Это происходит потому что когда функция вызывается как конструктор (то есть с оператором `new`), контекст в ней будет указывать на новосозданный объект.

## Приоритеты применения контекста
1. Оператор `new`
2. Методы `call()`, `bind()`, `apply()`
3. Вызов через точку
4. Window, undefined

## Стрелочные функции
Стрелочные функции (`() => {}`) имеют немного другое поведение с контекстом. Они относятся к `this` как к любой другое переменной, которая не объявлена в их области видимости и идут искать её значение вверх по цепочке областей видимости.

Пример 1:
```js
window.test = 2
const myFunc = () => console.log(this.test)
myFunc() // 2
```

Пример 2:
```js
const user = {
  name: 'Anna',
  superFunction: function () {
    const myFunc = () => console.log(this.name)
    myFunc()
  }
}
user.superFunction() // 'Anna'
```

Пример 3:
```js
const kottan = {
  city: 'Kyiv',
  language: 'JS',
  getCity: function() {
    return this.city
  },
  getLanguage: () => this.language
}

kottan.getCity() // "Kyiv"
kottan.getLanguage() // undefined
```

Пример 4:
```js
const logName = () => console.log(this.fullName)
logName.call({fullName: 'Mery Peterson'}) // undefined
```

# Практика

## Пример 1
```js
const kottan = {
  name: 'Kottan',
  city: 'Kyiv',
  language: 'JS'
}

Object.keys(kottan) // ['name', 'city', 'language']
Object.values(kottan) // ['Kottan', 'Kyiv', 'JS']
Object.entries(kottan) // [['name', 'Kottan'], ['city', 'Kyiv'], ['language', 'JS']]
```

## Пример 2
```js
const propertyName = 'name';

const user = {
  [propertyName]: 'John Doe'
};
console.log(user) // {name: 'John Doe'}
```

## Пример 3
```js
const userData = { name: 'Anna', age: 20 }
const { name, age: currentAge } = userData;
const user = { name, currentAge }
console.log(user) // {name: 'Anna', currentAge: '20'}
```

## Пример 4
```js
function sayName() {
  return this.name
}
const user1 = {
  name: 'Mary',
}
const user2 = {
  name: 'John'
}
const sayMarysName = sayName.bind(user1)
const sayJohnsName = sayMarysName.bind(user2)
console.log(sayJohnsName()) // Mary
```
## Пример 5
```js
var outer = {
  name: 'Joe',
  inner: {
    name: 'Jane',
    sayName: function() {
      console.log(this.name)
    }
  }
}
outer.inner.sayName() // Jane
outer.inner.sayName.call(outer) // Joe
```
