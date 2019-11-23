# v1
```go
func worker(cancel chan bool){
	for {
		select{
		default:
			fmt.Println("hello")
		case <-cancel:
			//quit
		}
	}
}

func main() {
	cancel:=make(chan bool)
	go worker(cancel)
	
	time.Sleep(time.Second)
	cancel<-true
}
```

这种方式退出会导致每创建一个goroutine都要增加chan的大小, 比较优雅的方式是使用cancel(chan), 他会广播给所有routine

# v2
当每个goroutine停止一般会进行一些清理工作, 但是main线程不会主动等待他们结束
```go 
func worker(wg *sync.WaitGroup, cancel chan bool){
	
	defer wg.Done()
	
	for{
		select{
		default:
			fmt.Println("hello")
		case <-cancel:
			return
		}
	}
	
}

func main() {
	cancel:=make(chan bool)
	var wg sync.WaitGroup
	for i:=0;i<10;i++{
		wg.Add(1)
		go worker(&wg, cancel)	
	}

	time.Sleep(time.Second)
	close(cancel)
	
	wg.Wait()
}
```

# v3
基于context实现
```go
func worker(ctx context.Context, wg *sync.WaitGroup) error{

	defer wg.Done()

	for{
		select{
		default:
			fmt.Println("hello")
		case <-ctx.Done():
			return ctx.Err()
		}
	}
}

func main() {

	ctx, cancel:=context.WithTimeout(context.Background(), 10*time.Second)
	
	var wg sync.WaitGroup
	for i:=0;i<10;i++{
		wg.Add(1)
		go worker(ctx, &wg)
	}

	time.Sleep(time.Second)

	cancel()
	wg.Wait()
}
```