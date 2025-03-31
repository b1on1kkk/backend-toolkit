## <a name="go_livecoding"></a>Лайвкодинг

### Яндекс:

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
