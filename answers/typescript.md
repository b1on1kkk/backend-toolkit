## <a name="utility_types"></a>Что такое Utility Types?

- **Awaited** - тип данных возвращаемых промисом
- **Partial** - все свойства становятся опциональными
- **Required** - все свойства обязательные
- **Readonly** - все свойства не могут быть перезаписаны
- **Record** - для коллекции одинаковых пар ключ-значение
- **Pick** - выбор отдельных свойств объекта
- **Omit** - убирает отдельные свойства
- **Exclude** - убирает из типа указанные свойства
- **Extract** - выбирает из типа указанные свойства
- **NonNullable** - оставляет в типе любые значения кроме null и undefined
- **ReturnType** - создает тип из типов return функции Type

## <a name="type_vs_interfaces"></a>Type vs Interface

### 1. Слияние интерфейсов

Интерфейсы поддерживают декларативное слияние, а псевдонимы типов нет. Объявив два или более интерфейса с одинаковыми идентификаторами (именами), мы получим один общий интерфейс:

```TypeScript
interface Order {
  id: string;
  createdAt: Date;
  items: string[];
}

interface Order {
  status: string;
}

interface Order {
  owner: string;
}

const myOrder: Order = {
  id: '31337',
  createdAt: new Date(),
  items: ['orange', 'banana'],
  status: 'created',
  owner: 'Michael Jackson',
}
```

`Interface` для объектов, а `Type` для различных типов, `Interface` может имплементивароваться классом.

---

### 2. Типы пересечения

В TypeScript эту задачу решает оператор амперсанд `(&)`.

```TypeScript
type OrderIdentifier = {
  id: string;
}

type OrderStatus = {
  status: string;
}

type Order = OrderIdentifier & OrderStatus;
```

Но и такое же можно провернуть с использованием `Interface`

```TypeScript
interface OrderIdentifier {
  id: string;
}

interface OrderStatus {
  status: string;
}

type Order = OrderIdentifier & OrderStatus;
```

---

### 3. Интерфейсы и классы

Интерфейсы особенно удобны при использовании объектно-ориентированного подхода. Сначала проектируется интерфейс, а потом классы, которые его имплементируют:

```TypeScript
interface Cat {
  meow: () => void;
}

class Tiger implements Cat {
  meow() {
    console.log('meow-meow-bark');
  }
}
```

Можно и заменить на `Type`:

```TypeScript
type Cat = {
  meow: () => void;
}
```

---

### 4. Наследование интерфейсов

```TypeScript
interface Cat {
  meow: () => void;
}

interface FastCat extends Cat {
  run: () => void;
}

class Tom implements FastCat {
  meow() {
    console.log('meow');
  }

  run() {
    console.log('run');
  }
}
```

---

### 5. Кортежи

```TypeScript
type Developer = [string, string, number];
const ivan: Developer = ['Ivan', 'Ivanov', 33];
```

Объявить новый кортеж с помощью `Interface` нельзя

## <a name="never_and_unknown"></a>Зачем нужен never, unknown?

- **Never** - признак значений, которых никогда не будет. Или, признак для
  функций, которые никогда не вернут значения, то ли по причине ее зацикленности? например, бесконечный цикл, то ли по причине ее прерывания.

- **Unknown** – безопасная версия типа any. Тип unknown позволяет заранее обнаруживать ошибки времени выполнения (на стадии компиляции).

## <a name="generics"></a>Дженерики в TypeScript

[Дженерики или Generic Types](https://habr.com/ru/companies/tbank/articles/588655/) - обобщенные типы. Они нужны для описания похожих, но отличающихся какими-то характеристиками типов.
