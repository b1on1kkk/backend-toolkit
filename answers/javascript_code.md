## <a name="javascript_code"></a>Разбор задачек на JavaScript

### Что выведет этот код:

```JavaScript
for (var i = 1; i < 5; i++) {
    setTimeout (() => console.log(i), 1000);
}
```

### Ответ:

```JavaScript
5, 5, 5, 5
```

### Разбор почему так:

Переменная `var` в JavaScript имеет функциональную область видимости, а не блочную. Переменная `i` создаётся в области видимости функции, содержащей цикл. Это означает, что одна и та же переменная `i` используется на каждой итерации цикла.

---

### Правильно ли работает этот код?

```JavaScript
const userName = "Sinichkin"

if (userName === "Ivanov" || 'Petrov' || 'Sidorov') {
    console.log ("Access granted")
} else {
    console.log("Forbidden")
}
```

### Ответ:

```JavaScript
"Access granted"
```

### Разбор почему так:

Такой ответ будет всегда, не зависимо от переданного `userName`. Проверяет, равен ли `userName` строке "Ivanov". Если да, то эта часть возвращает `true`. В JavaScript любое ненулевое или непустое значение, например строка 'Petrov', интерпретируется как `true`, потому что она не пустая и словие дальше не проверяется.

---

### Что выведет этот код:

```JavaScript
const arr = [1,2,3]
arr [10] = 99
console.log(arr.length)
```

### Ответ:

```JavaScript
11
```

### Разбор почему так:

Массив расширится до 11 элементов, заполняя все предыдущие, не используемые, индексы значениями undefined.

---

### Что выведет этот код:

```JavaScript
function makeGroup () {
    let people = [];

    let i = 0;
    while (i < 10) {
        let man = function () {
            alert(i);
        };
        people.push(man);

        i++;
    }

    return people;
}

let group = makeGroup();

group[0]();
group[5]();
```

### Ответ:

```JavaScript
10 10
```

### Разбор почему так:

Функции в массиве ссылаются на одну и ту же переменную `i`, но не создают ее копию, а просто используют `i` из внешней области видимости.

---

### Что выведет этот код:

```JavaScript
function updateProfile(person){
    person = {
        name: "Peter"
    }
}

function proccess(){
    let person = {name: "Mike"};

    updateProfile(person);

    console.log(person);
}
proccess(); //
```

### Ответ:

```JavaScript
{ name: "Mike" }
```

### Разбор почему так:

В JavaScript объекты передаются в функции по ссылке, но только на уровне изменения свойств. Если же внутри функции присвоить аргументу новый объект, то это разрывает связь с исходной переменной.

---

### Что выведет этот код:

```JavaScript
(function question(){
    const p = Promise.reject();

    p
        .catch(()=>console.log(5)) // 5
        .then(()=>console.log(6)) // 6
        .catch(()=>console.log(7)) // не выполнится

    p
        .then(()=>console.log(10)) // не выполнится
        .catch(()=>console.log(7)) // 7
        .then(()=>console.log(12)) // 12
})();
```

### Ответ:

```JavaScript
5
6
7
12
```

### Разбор почему так:

Если очередной `then` вернул промис, то далее по цепочке будет передан не сам этот промис, а его результат. По чейнингу идет просто вниз до конца, пока `then` что то возвращает
