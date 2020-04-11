# GO routines patterns to merge data from different asynchronous calls

Say you have an API that collects data multiple data sources and returns a single resultset, how would you model such code in go.
Once way to do it is using synchronous calls to multiple data sources and cllecting the data into a single data structure and returning this data structure as a result.
For example:

```go
func main(){
finalResult:=[]Result{}

result1, err:= getDataFromAPI1()
if err!= nil{
    panic(err)
}
append(finalResult,data1)

result2, err:= getDataFromDataAPI2()
if err!= nil{
    panic(err)
}
append(finalResult,data2)

result3, err:= getDataFromDataAPI3()
if err!= nil{
    panic(err)
}
append(finalResult,data3)

fmt.Print(finalResult)
}
```
This technique works well, however there is one caveat, if these data sources are over the network any delay in these network calls with addup and increase the total time of execution of the program.
To solve this problem we would use go routines, the three APIs would be called inside a three seperate go routine this would reduce the overall time of execution since any network delay would overlap with other API calls. 
But how do you merge the results? One way I have seen programmers go it is using global variable and use mutex to merge the resultset
for example.
```go
func main() {
	finalResult:=[]Result{}
	var mux sync.Mutex
	var wg sync.WaitGroup
	wg.Add(3)
	go func() {
		defer wg.Done()
    result1, err:= getDataFromAPI1()
    if err!= nil{
      panic(err)
    }
    mux.Lock()
    append(finalResult,result1)
    mux.Unlock()

	}()
	go func() {
		defer wg.Done()
    result2, err:= getDataFromAPI2()
    if err!= nil{
      panic(err)
    }
    mux.Lock()
    append(finalResult,result2)
    mux.Unlock()

	}()
	go func() {
		defer wg.Done()
    result1, err:= getDataFromAPI3()
    if err!= nil{
      panic(err)
    }
    mux.Lock()
    append(finalResult,result3)
    mux.Unlock()

	}()
	wg.Wait()
	fmt.Print(finalResult)
}

```
This code works well, but still has problems
