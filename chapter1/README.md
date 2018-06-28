# Chapter 1

## Race Conditions

* race condition
  * two or more operations must execute in the correct order
  * but the order is not guaranteed.
  * Result: output is **indeterminate**

* data race
  * one concurrent operation attempts to read a variable
  * another concurrent operation is attempting to write to the same variable
  * Result: output is **indeterminate**

```go
var data int
go func() {
    data++
}()
if data == 0 {
    fmt.Println("the value is ", data)
} else {
    fmt.Println("the value is not 0 but ", data)
}
// What's the output?
// Does it have race condition?
// Does it have data race?
```

### Atomicity

* atomic
  * intermediate state is indivisible
  * operation is uninterruptible

```go
// Are these atomic?
i = 5
i = a
i++
```

```bash
## pseudocode assembly

# i = 5
MOV  i, 5

# i = a
MOV AX, a
MOV i,  AX

# i++
MOV  AX, i
INC  AX
MOV  i,  AX
```

### Memory Access Synchronization

```go
var memoryAccess sync.Mutex
var data int
go func() {
    memoryAccess.Lock()
    data++
    memoryAccess.Unlock()
}()
memoryAccess.Lock()
if data == 0 {
    fmt.Println("the value is ", data)
} else {
    fmt.Println("the value is not 0 but ", data)
}
memoryAccess.Unlock()
// What's the output?
// Does it have race condition?
// Does it have data race?
```

## Deadlocks

* one in which all concurrent processes are waiting on one another
* the program will never recover without outside intervention

```go
type value struct {
    mu    sync.Mutex
    value int
}

var wg sync.WaitGroup
printSum := func(v1, v2 *value) {
    defer wg.Done()
    v1.mu.Lock() 1
    defer v1.mu.Unlock() 2

    time.Sleep(2*time.Second) 3
    v2.mu.Lock()
    defer v2.mu.Unlock()

    fmt.Printf("sum=%v\n", v1.value + v2.value)
}

var a, b value
wg.Add(2)
go printSum(&a, &b)
go printSum(&b, &a)
wg.Wait()
```

### Coffman Conditions

A deadlock situation on a resource can arise if and only if all of the following conditions hold simultaneously in a system

* Mutual Exclusion
  * A concurrent process holds exclusive rights to a resource at any one time
* Wait For Condition
  * A concurrent process must simultaneously hold a resource and be waiting for an additional resource
* No Preemption
  * A resource held by a concurrent process can only be released by that process
* Circular Wait
  * A concurrent process (P1) must be waiting on a chain of other concurrent processes (P2), which are in turn waiting on it (P1)

## Livelocks

* programs that are actively performing concurrent operations
* these operations do nothing to move the state of the program forward
* Reason: two or more concurrent processes attempting to prevent a deadlock without coordination

Example: 

* You are in a hallway walking toward another person.
* She moves to one side to let you pass, but you’ve just done the same.
* So you move to the other side, but she’s also done the same.
* This is going on forever.

## Starvation

* any situation where a concurrent process cannot get all the resources it needs to perform work

Example: a greedy process holds a resource for a long time so other processes are blocked forever

## Conclusion

* program stops in **indeterminate** state
  * race Condition
    * two or more operations must execute in the correct order
    * but the order is not guaranteed
  * data race
    * one concurrent operation attempts to read a variable
    * another concurrent operation is attempting to write to the same variable
* program don't stop
  * deadlock
    * all processes are blocked, the program hangs forever
  * livelock
    * No processes blocked but they run into infinite loops. The program is still running but unable to make further progress
  * starvation
    * only one process is running, and other processes are waiting forever
