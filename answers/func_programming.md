## <a name="arity"></a>Arity (арность)

Количество аргументов функции. Сложение, к примеру, принимает два аргумента, поэтому это бинарная функция, или функция, у которой арность равна двум.

```JavaScript
const sum = (a, b) => a + b
```

## <a name="higher_order_functions"></a>Функции высшего порядка (Higher Order Functions) в JavaScript

Функция высшего порядка — это функция, возвращающая другую функцию или принимающая другую функцию в качестве аргумента.

## <a name="currying"></a>Currying (каррирование)

Преобразование функции с множеством аргументов в набор вложенных функций с одним аргументом. При вызове каррированной функции с передачей ей одного аргумента, она возвращает новую функцию, которая ожидает поступления следующего аргумента. Новые функции, ожидающие следующего аргумента, возвращаются при каждом вызове каррированной функции — до тех пор, пока функция не получит все необходимые ей аргументы.

Иными словами - это процесс превращения функции с несколькими аргументами в функцию с меньшей арностью.

Обычная функция:

```JavaScript
function multiply(a, b, c) {
    return a * b * c;
}

multiply(1,2,3); // 6
```

Каррированная функция:

```JavaScript
function multiply(a) {
    return (b) => {
        return (c) => {
            return a * b * c
        }
    }
}

multiply(1)(2)(3) // 6
```

---

Если конструкция вида `multiply(1)(2)(3)` кажется вам не слишком понятной, распишем её в таком виде:

```JavaScript
const mul1 = multiply(1);
const mul2 = mul1(2);
const result = mul2(3);

console.log(result); // 6
```

## <a name="pure"></a>Pure (чистая)

Функция является чистой, если возвращаемое ей значение определяется исключительно вводными значениями, и функция не имеет побочных эффектов.

`Pure` function:

```JavaScript
const greet = (name) => 'Hi, ' + name

greet('Brianne') // 'Hi, Brianne'
```

**Not** `Pure` function:

```JavaScript
let greeting

const greet = () => {
  greeting = 'Hi, ' + window.name
}

greet() // "Hi, Brianne"
```

## <a name="side_effects"></a>Side effects (побочные эффекты)

Функция не только возвращает результат, но и делает что-то кроме. Например, выводит что-то на экран, записывает в файл.

Function with `Side effect`:

```JavaScript
let user = { name: "Alice" };

function changeName(obj) {
    obj.name = "Bob";
}

changeName(user);

console.log(user.name);  // Bob
```

Function without `Side effect`:

```JavaScript
function add(a, b) {
    return a + b;
}

add(2, 3); // 5

```
