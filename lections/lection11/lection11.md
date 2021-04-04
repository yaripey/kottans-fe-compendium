# Лекция 11. Итераторы и генераторы. Map, Set

[Назад](../../README.md)

# Итераторы

Рассмотрим пример:

```jsx
const myArr = [1, 2, 3];
for (item of myArr) {
  console.log(item);
}
// 1
// 2
// 3
```

Оператор `for ... of` ожидает от нас некоторый _iterable object_, то есть объект, который можно _перебрать_. Это массивы, строки, и т.д.

В данном примере, `for ... of` это **data consumer**, то есть сущность, которая перебирает данные, а `myArr` это **data producer,** то есть сущность, которая предоставляет данные для перебора.

Другой пример data consumer'а это spread operator (`...`). Он ожидает какую-то итерируемую сущность, берёт каждую её частицу и копирует туда, где указан.

Ещё один пример это `Promise.all()`.

Условием итерируемой сущности является наличие у неё свойства `Symbol(Symbol.iterator)`.

Какой-нибудь массив является `iterable` сущностью, на которой реализована сущность `iterator`. Эта сущность и будет по одному отдавать нам элементы массива при переборе.

Сам метод `Symbol.iterator` возвращает объект-итератор. Этот объект имеет метод `next()` который возвращает объект с двумя полями - `value`, в котором будет находиться значение из итерируемой сущности, а в поле `done` будет храниться булевое значение, которое указывает на то, остались ли ещё элементы в итерируемой сущности. Пример:

```jsx
const iterator = [1, 2, 3][Symbol.iterator]();
iterator.next(); // { value: 1, done: false }
iterator.next(); // { value: 2, done: false }
iterator.next(); // { value: 3, done: false }
iterator.next(); // { value: undefined, done: true }
```

Мы так же можем создать свою собственную итерируемую сущность следующим образом:

```jsx
const myIterable = {
  [Symbol.iterator]() {
    let currentValue = 0;
    const maxValue = 10;

    return {
      next() {
        if (currentValue > maxValue) {
          return {
            value: undefined,
            done: true,
          };
        }
        return {
          value: currentValue++,
          done: false,
        };
      },
    };
  },
};
```

# Генераторы

Генератор, это функция, выполнение которой можно приостановить. Мы создаём функцию с указанием звёздочки (`*`) после её названия, а внутри мы можем использовать ключевое слово `yield`. Это что-то вроде `return`, только помимо возвращения значения это ключевое слово так же останавливает выполнение функции.

```jsx
function* myGenerator() {
  yield 1;
  yield 2;
  yield 3;
}
```

Функция-генератор возвращает итератор. Таким образом:

```jsx
for (item of myGenerator()) {
  console.log(item);
}
// 1
// 2
// 3
```

## Async/await vs generators

Мы можем сравнить синтаксис написания функции с использованием ключевых слов `async` / `await` и функции генератора.

Мы будем выполнять следующие асинхронные функции:

```jsx
function doSomeAsync1() {
  return Promise.resolve("result 1");
}
function doSomeAsync2() {
  return Promise.resolve("result 2");
}
function doSomeAsync3() {
  return Promise.resolve("result 3");
}
```

### Async/await

```jsx
async function doAsyncStuff() {
  const result1 = await doSomeAsync1();
  console.log(result1);
  const result2 = await doSomeAsync2();
  console.log(result2);
  const result3 = await doSomeAsync3();
  console.log(result3);
}

doAsyncStuff();
// result 1
// result 2
// result 3
```

### Генератор

```jsx
function* asyncGenerator() {
  const result1 = yield doSomeAsync1();
  console.log(result1);
  const result2 = yield doSomeAsync2();
  console.log(result2);
  const result3 = yield doSomeAsync3();
  console.log(result3);
}

const gen = asyncGenerator();
gen.next();
// { value: Promise, done: false }
gen.next().value.then(console.log);
// result 2
```

В таком случае мы получаем итератор из которого будем получать промисы. Чтобы их обработать, нам понадобится внешний код, который будет ждать их выполнения. Например, мы можем написать функцию-раннер:

```jsx
function* myGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

function runner(generatorFunc) {
  const gen = generatorFunc();
  function run() {
    const result = gen.next();
    if (result.done) {
      return result.value;
    }
    return Promise.resolve(result.value).then(run);
  }
  return run();
}

runner(myGenerator);
// 1
// 2
// 3
```

**Важно**, мы можем при вызове метода `.next` при использовании генераторов, передавать в него какое-то значение, которое попадёт в функцию генератор, когда она продолжит своё выполнение.

# Map (dictionary, hash table)

Map - это структура данных, которая представляет из себя набор из соответствий _ключ - значение_. Немного похоже на объект, с отличием в том, что у объекта ключи - это строки либо символы. У Map ключём может быть любой тип данных.

```jsx
const map = new Map();
map.set("key", "value");
map.get("key"); // "value"
```

Помимо использования метода `.set` мы можем наполнять map не поштучно, а передавая в него iterable сущность в момент создания (частицей такой сущности должен быть массив с соответствием ключ-значение):

```jsx
const myMapFromArr = new Map(["one", 1], ["two", 2]);
```

У Map так же есть свойство `.size`, которое отображает количество элементов в map.

Map является iterable сущностью, так что её можно перебрать при помощи, например, `for ... of`.

В отличие от объектов, Map содержит в себе только те значения, которые мы ему передали. То есть никаких дополнительных методов или значений прототипов у Map не будет.

# Set

Структура данных, подразумевающая набор **строго уникальных** значений. Интерфейс похож на Map, можно добавлять значения при создании (правда, это должны быть просто значения, а не соответствия ключ-значение, как при Map), а так же пользоваться методами `.add`(не `.set`) и `.get`. При добавлении нового значения, Set сам для на проверяет себя на факт существования такого же значения, и если находит его, игнорирует добавление. То есть в Set никогда не может быть дубликатов.

```jsx
const set = new Set();
set.add(1);
set.add(3);
```

У Set так же есть `.size`.

У Set есть метод `.clear()`, очищающий набор.

У Set есть метод `.has()`, проверяющий наличие элемента в наборе (возвращает булевое значение).

Set так же можно перебирать.

Set'ом можно пользоваться как фильтром для повторяющихся значений, например:

```jsx
Array.from(new Set([1, 2, 3, 4, 2, 1, 2, 3, 4, 2, 1]));
// [1, 2, 3, 4]
```

# WeakMap и WeakSet

В JS мы не можем управлять памятью как в некоторых других языках программирования. Подчисткой неиспользуемой занятой памяти в JS занимается _сборщик мусора_. Он сам решает когда и какие вещи удалять. Существует 2 варианта ссылок на данные - weak binding и strong binding.

Strong binding подразумевает что пока есть хоть одна ссылка на объект, он будет существовать в памяти.

WeakMap и WeakSet будут хранить в себе ссылки на объекты только до тех пор, пока объект не будет удалён. Таким образом они не замыкают объект и не создают утечек памяти.
