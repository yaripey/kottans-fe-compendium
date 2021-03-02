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
