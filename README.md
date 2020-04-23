# timingwheel
分层时间轮的Golang实现, 基于[RussellLuo/timingwheel](https://github.com/RussellLuo/timingwheel) 改造

## Installation

```bash
$ go get -u github.com/golearnku/timingwheel
```


## Design

`timingwheel` is ported from Kafka's [purgatory][1], which is designed based on [Hierarchical Timing Wheels][2].

中文博客：[层级时间轮的 Golang 实现][3]。


## Documentation

For usage and examples see the [Godoc][4].

## AfterFunc 
```go
package main

import (
	"fmt"
	"time"

	"github.com/golearnku/timingwheel"
)

func main()  {
	tw := timingwheel.NewTimingWheel(time.Millisecond, 20)
	tw.Start()
	defer tw.Stop()

	t := tw.AfterFunc("100",time.Second, func() {
		fmt.Println("The timer fires")
	})

	<-time.After(900 * time.Millisecond)
	// Stop the timer before it fires
	t.Stop()
}
```

```go
package main

import (
	"fmt"
	"time"

	"github.com/golearnku/timingwheel"
)

func main()  {
	tw := timingwheel.NewTimingWheel(time.Millisecond, 20)
	tw.Start()
	defer tw.Stop()

	exitC := make(chan time.Time, 1)
	tw.AfterFunc("100",time.Second * 2, func() {
		fmt.Println("The timer fires")
		exitC <- time.Now().UTC()
	})

	<-exitC
}
```

## ScheduleFunc
```go
package main

import (
	"fmt"
	"time"

	"github.com/golearnku/timingwheel"
)

type EveryScheduler struct {
	Id       int
	Interval time.Duration
}

func (s *EveryScheduler) Next(prev time.Time) time.Time {
	return prev.Add(s.Interval)
}

func main() {
	tw := timingwheel.NewTimingWheel(time.Millisecond, 20)
	tw.Start()
	defer tw.Stop()

	i := 0

	tw.ScheduleFunc("100", &EveryScheduler{1, time.Second * 2}, func() {
		i++
		fmt.Println("The timer fires")
		fmt.Println(i)
		//exitC <- time.Now().UTC()
	})

	for {
		select {
		case <-time.After(time.Millisecond * 300):
			i += 100
		case <-time.After(time.Second * 5):
			tw.Remove("100")
			return
		}
	}
}
```

## Benchmark

```
$ go test -bench=. -benchmem
goos: darwin
goarch: amd64
pkg: github.com/golearnku/timingwheel
BenchmarkTimingWheel_StartStop/N-1m-8         	 2502430	       459 ns/op	     134 B/op	       4 allocs/op
BenchmarkTimingWheel_StartStop/N-5m-8         	 2732517	       522 ns/op	     147 B/op	       4 allocs/op
BenchmarkTimingWheel_StartStop/N-10m-8        	 2098280	       493 ns/op	      70 B/op	       1 allocs/op
BenchmarkStandardTimer_StartStop/N-1m-8       	 7412431	       232 ns/op	      81 B/op	       1 allocs/op
BenchmarkStandardTimer_StartStop/N-5m-8       	 4012328	       290 ns/op	      84 B/op	       1 allocs/op
BenchmarkStandardTimer_StartStop/N-10m-8      	 5873055	       280 ns/op	      86 B/op	       1 allocs/op
PASS
ok  	github.com/golearnku/timingwheel	80.234s
```


## License

[MIT][5]


[1]: https://www.confluent.io/blog/apache-kafka-purgatory-hierarchical-timing-wheels/
[2]: http://www.cs.columbia.edu/~nahum/w6998/papers/ton97-timing-wheels.pdf
[3]: http://russellluo.com/2018/10/golang-implementation-of-hierarchical-timing-wheels.html
[4]: https://godoc.org/github.com/golearnku/timingwheel
[5]: http://opensource.org/licenses/MIT