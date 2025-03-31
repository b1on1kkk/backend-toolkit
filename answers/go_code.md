## <a name="go_code"></a>Разбор задачек на Golang

### Что выведет этот код:

```Golang
package main

import "fmt"

func main() {
  a := [5]int{1, 2, 3, 4, 5}
  t := a[3:4:4]

  fmt.Println(t[0])
}
```

### Ответ:

```Golang
4
```

### Разбор почему так:

Такой синтаксис позволяет задать `capacity` для полученного под-слайса, который будет равен _"последний элемент минус первый элемент из выражения в квадратных скобках"_, т.е. из примера выше он будет равен 1

Если бы выражение имело вид `a[3:4:5]`, то cap была бы равна `двум`: (5 — 3 = 2). Но при этом на сами данные он не влияет.

[Slicing a slice with slice](https://stackoverflow.com/questions/27938177/golang-slice-slicing-a-slice-with-sliceabc/27938683#27938683).

---

### Что выведет этот код:

```Golang
package main

import "fmt"

func main() {
	nums := []int{1, 2, 3}

	addNum(nums[0:2])
	fmt.Println(nums) // -

	addNums(nums[0:2])
	fmt.Println(nums) // -
}

func addNum(nums []int) {
	nums = append(nums, 4)
}

func addNums(nums []int) {
	nums = append(nums, 5, 6)
}
```

### Ответ:

```Golang
[1 2 4]

[1 2 4]
```

### Разбор почему так:

Создается слайс `nums`. Создается подслайс `nums[0:2]` слайса `nums` со значениями `[1, 2]`, с `len == 2` и `cap == 3`, и передается в функции `addNum` и `addNums` по значению: структура слайса копируется.

Что происходит в функции `addNum`. Поскольку `cap` переданного слайса больше его длинны, у нас есть возможноть добавить новое значение. Добавляем, перезаписывается 3 на 4, поскольку базовый массив в структуре слайса передается по сслыке.

```Golang
// передали по значению (структура скопирована), ссылаемся на базовый массив
func addNum(nums []int) {
	nums = append(nums, 4) // len == 2, cap == 3, есть возможность записать значение
}
```

Что происходит в функции `addNums`. Поскольку в данном случае мы хотим добавить уже два значения, но `cap` такой возможности не дает — аллоцируем новую память для НОВОГО слайса. При аллоцировании новой памяти, nums `начинает` ссылаться на другой базовый массив, не изменяя переданный

```Golang
// передали по значению (структура скопирована), ссылаемся на базовый массив
func addNums(nums []int) {
	nums = append(nums, 5, 6) // при добавлении двух значений происходит создание нового массива с новой ссылкой, которая никак не меняет переданный слайс
}
```

[Разбор слайсов](https://www.youtube.com/watch?v=10LW7NROfOQ).

---

### Что выведет этот код:

```Golang
package main

import "fmt"

type MyError {}

func (m *MyError) Error() string {
	return "boom"
}

func fail() error {
	var err *MyError = nil
	return err
}

func main() {
	err := fail()
	if err == nil {
		fmt.Println("No error")
	} else {
		fmt.Println("Error occured")
	}
}
```

### Ответ:

```Golang
"Error occured"
```

### Разбор почему так:

Поскольку в Go тип `interface` имеет под капотом структуру с двумя значениями: указатель на значение и указатель на тип, мы присвоили переменной `err` какой то тип, в данном случае указатель на структуру `MyError` => он уже не nil и поэтому условие выполняется: переменная не пустая.

---

### Лайвкодинг. Яндекс:

```Golang
/*
Есть приложение с микросервисной архитектурой.
Микросервис можно абстрагировать с помощью интерфейса Backend.
Для доступа к одному экземпляру микросервиса можно использовать
тип BackendImpl, который уже реализован.

Для каждого микросервиса есть несколько десятков запущенных
экземпляров, каждый из которых доступен по своему адресу addr.
Однако отдельные экземпляры микросервиса ненадежны:
они могут падать, быть недоступными либо перегруженными.
Поэтому вам нужно реализовать тип Balancer, который также реализует
интерфейс Backend и осуществляет client-side балансировку нагрузки
между экземплярами микросервиса.

Задание:
реализовать Balancer, который будет распределять нагрузку с помощью алгоритма Round-Robin
*/

package main

import (
	"context"
	"errors"
	"fmt"
	"sync"
	"time"
)

type Request interface{}

type Response interface{}

type Backend interface {
	Invoke(ctx context.Context, req Request) (Response, error)
}

//
type BackendImpl struct {
	addr string
}

func (b *BackendImpl) Invoke(ctx context.Context, req Request) (Response, error) {
	return nil, nil
}

func NewBackend(addr string) *BackendImpl {
	return &BackendImpl{addr: addr}
}
//

//
type Balancer struct {
	backends   []Backend
	currentIdx int
	mutex      *sync.Mutex
}

func (b *Balancer) Invoke(ctx context.Context, req Request) (Response, error) {
	b.mutex.Lock()
	defer b.mutex.Unlock()

	if len(b.backends) == 0 {
		return nil, errors.New("no available backends")
	}

	fmt.Println("current idx: ", b.currentIdx)

	resp, err := b.backends[b.currentIdx].Invoke(ctx, req)

	b.currentIdx++

	if b.currentIdx >= len(b.backends) {
		b.currentIdx = 0
	}

	if err != nil {
		return nil, err
	}

	return resp, nil
}
//

// addrs содержат адреса всех балансируемых экземпляров
func NewBalancer(addrs []string) *Balancer {
	if len(addrs) == 0 {
		return &Balancer{backends: nil}
	}

	backends := make([]Backend, 0, len(addrs))
	var mutex sync.Mutex

	for _, addr := range addrs {
		backends = append(backends, NewBackend(addr))
	}

	return &Balancer{
		backends: backends,
		mutex:    &mutex,
	}
}

// юзаем паттерн семафоры, для распределения нагрузки
type Semaphore struct {
	sema chan struct{}
}

func NewSemaphore(maxReq int) *Semaphore {
	return &Semaphore{
		sema: make(chan struct{}, maxReq),
	}
}

func (s *Semaphore) Acquire() {
	s.sema <- struct{}{}
}

func (s *Semaphore) Release() {
	<-s.sema
}
//

func main() {
	addrs := []string{"127.0.0.1:8001", "127.0.0.1:8002", "127.0.0.1:8003", "127.0.0.1:8004", "127.0.0.1:8005", "127.0.0.1:8006", "127.0.0.1:8007", "127.0.0.1:8008"}
	balancer := NewBalancer(addrs)
	semaphore := NewSemaphore(3)

	ctx := context.Background()
	ctxDeadline, cancel := context.WithTimeout(ctx, time.Second*1)
	defer cancel()

	var wg sync.WaitGroup

loop:
	for {
		select {
		case <-ctxDeadline.Done():
			break loop
		default:
			semaphore.Acquire()
			wg.Add(1)
			go func() {
				defer semaphore.Release()
				defer wg.Done()

				resp, err := balancer.Invoke(ctxDeadline, struct{}{})

				if err != nil {
					fmt.Println("Ошибка:", err.Error())
				} else {
					fmt.Println("Ответ получен:", resp)
				}
			}()
		}
	}

	wg.Wait()
}
```
