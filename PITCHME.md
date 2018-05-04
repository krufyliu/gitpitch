---
@title[介绍]
# 初识Context

<br>

#### 2018-05-03 可乐

---
@title[context包]
### context package
<br>
<br>
顾名思义

---

- golang.org/x/net/context    
很多第三方库都依赖于这个包
- [context](https://blog.golang.org/context)     
1.7 之后引入标准库

---
@title[Context的作用]
### 为什么需要Context

<br>

- 控制goroutine执行
- 长请求超时限制
- 上下文传递

---
@title[接口定义]
### 接口定义
<br>
```golang
// A Context carries a deadline, cancelation signal, and request-scoped values
// across API boundaries. Its methods are safe for simultaneous use by multiple
// goroutines.
type Context interface {
    // Done returns a channel that is closed when this Context is canceled
    // or times out.
    Done() <-chan struct{}

    // Err indicates why this context was canceled, after the Done channel
    // is closed.
    Err() error

    // Deadline returns the time when this Context will be canceled, if any.
    Deadline() (deadline time.Time, ok bool)

    // Value returns the value associated with key or nil if none.
    Value(key interface{}) interface{}
}
```
---
@title[context提供的API]
### Context创建API
<br>
```golang
// Background returns a non-nil, empty Context. It is never canceled, has no values, 
// and has no deadline. It is typically used by the main function, initialization, 
// and tests, and as the top-level Context for incoming requests.
func Background() Context
// TODO returns a non-nil, empty Context. Code should use context.
// TODO when it's unclear which Context to use or it is not yet available (because the surrounding function has not yet been extended to accept a Context parameter). 
// TODO is recognized by static analysis tools that determine whether Contexts are propagated correctly in a program.
func TODO() Context
```
---
```golang 
 func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
 func WithDeadline(parent Context, d time.Time) (Context, CancelFunc)
 func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
 func WithValue(parent Context, key, val interface{}) Context
```
---
@title[Context的特点]
### Context的特点
- 是不可变的(immutable)树节点
- Cancel 一个节点，会连带 Cancel 其所有子节点 （从上到下）
- Context values 是一个树节点
- Value 查找是回溯树的方式 （从下到上）
---
### 示例 Context 链
```golang
ctx1 := context.Background()
ctx2, _ := context.WithCancel(ctx1)
ctx3, _ := context.WithTimeout(ctx2, time.Second * 5)
ctx4, _ := context.WithTimeout(ctx3, time.Second * 3)
ctx5, _ := context.WithTimeout(ctx3, time.Second * 6)
ctx6 := context.WithValue(ctx5, "userID", 12)
```
---
![](asserts/images/context-chain-1.jpg)

---
@title[context在代码库中应用]
### context在代码库中应用

- net/http
- database/sql
- golang.org/x/sync/errgroup

---
@title[如何集成到我们的API中]
### 如何集成到我们的API中
- 如果有 Context，将其作为第一个变量
- 结构体方式,即作为结构体的一个变量，如http.Request

---
### 完