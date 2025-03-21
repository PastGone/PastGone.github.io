**初探**
------

我们先初步介绍以下`net/http`包的使用，通过`http.HandleFunc()`和`http.ListenAndServe()`两个函数就可以轻松创建一个简单的Go web服务器，示例代码如下:

```go
 package main
 ​
 import (
     "fmt"
     "net/http"
 )
 ​
 func hello(w http.ResponseWriter, r *http.Request) {
     fmt.Fprintf(w, "Hello!")
 }
 ​
 func main() {
     http.HandleFunc("/hello", hello)
     http.ListenAndServe(":8080", nil)
 }
```

在上面的代码中,`main()`函数通过代码`http.ListenAndServe(":8080“,nil)`启动一个8080端口的服务器。

此时在网页中输入`http://localhost:8080/hello`，就可以得到Hello！的字符串。

`ListenAndServe()`函数有两个参数，当前监听的端口号和事件处理器Handler。

事件处理器的Handler接口定义如下：

```go
 type Handler interface {
     ServeHTTP(ResponseWriter, *Request)
 }
```

只要实现了这个接口，就可以实现自己的handler处理器。Go语言在net/http包中已经实现了这个接口的公共方法：

```go
 type HandlerFunc func(ResponseWriter, *Request)
 ​
 // ServeHTTP calls f(w, r).
 func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
     f(w, r)
 }
```

如果ListenAndServe()传入的第一个参数地址为空，则服务器在启动后默认使用`http://127.0.0.1:8080`地址进行访问；如果这个函数传入的第二个参数为nil，则服务器在启动后将使用默认的多路复用器`DefaultServeMux`。

多路复用器的概念在后面再进行详解。

用户可以通过Server结构体对服务器进行更详细的配置，包括设置地址，为请求读取操作设置超过时间等等。

```go
 s := &http.Server{
     Addr:           ":8082",
     Handler:        myHandler,
     ReadTimeout:    10 * time.Second,
     WriteTimeout:   10 * time.Second,
     MaxHeaderBytes: 1 << 20,
 }
```

Go web服务器的请求和响应流程如下：

<img src="https://pic4.zhimg.com/v2-c4820c4e859e95cc634c64bf9b0b0fc3\_b.jpg" data-caption="" data-size="normal" data-rawwidth="996" data-rawheight="706" class="origin\_image zh-lightbox-thumb" width="996" data-original="https://pic4.zhimg.com/v2-c4820c4e859e95cc634c64bf9b0b0fc3\_r.jpg"/>

![](https://pic4.zhimg.com/v2-c4820c4e859e95cc634c64bf9b0b0fc3_r.jpg)

  

响应流程如下：

1.  客户端发送请求
2.  服务器的多路复用器收到请求
3.  多路复用器根据请求的URL找到注册的处理器，将请求交由处理器处理
4.  处理器执行程序逻辑，与数据库交互
5.  调用模板引擎选择模板
6.  服务器端将数据通过Http响应返回给客户端
7.  客户端拿到数据呈现给用户

**接受请求**
--------

### **ServeMux与DefaultServeMux**

多路复用器的基本原理：根据请求的URL地址找到对应的处理器，调用处理器对应的`ServeHTTP()`方法处理请求。

DefaultServeMux是net/http包的默认多路复用器，其实就是ServeMux的一个实例。

```go
 //Go source code
 var DefaultServeMux = &defaultServeMux
 var defaultServeMux ServeMux
```

HandleFunc()函数用于为指定的URL注册一个处理器。HandleFunc()处理器函数会在内部调用DefaultServeMux对象对应的方法，其内部实现：

```go
 // HandleFunc registers the handler function for the given pattern
 // in the DefaultServeMux.
 // The documentation for ServeMux explains how patterns are matched.
 func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
     DefaultServeMux.HandleFunc(pattern, handler)
 }
```

由此看出，我们也可以使用默认多路复用器注册多个处理器，达到与处理器一样的作用。

### **总结**

我们通过以上分析大概明白了如何通过默认多路复用器创建自己的服务器。下面我们来检验这一点：

```go
 package main
 ​
 import (
     "fmt"
     "net/http"
 )
 ​
 //定义多个处理器
 type handle1 struct{}
 func (h1 *handle1) ServeHTTP(w http.ResponseWriter, r *http.Request) {
     fmt.Fprintf(w, "handle one")
 }
 ​
 type handle2 struct{}
 func (h2 *handle2) ServeHTTP(w http.ResponseWriter, r *http.Request) {
     fmt.Fprintf(w, "handle two")
 }
 ​
 func main() {
     handle1 := handle1{}
     handle2 := handle2{}
     //Handler:nil表明服务器使用默认的多路复用器DefaultServeMux
     s := &http.Server{
         Addr:    "127.0.0.1:8080",
         Handler: nil,
     }
     
     //Handle函数调用的是多路复用器DefaultServeMux.Handle方法
     http.Handle("/handle1", &handle1)
     http.Handle("/handle2", &handle2)
 ​
     s.ListenAndServe()
 }
```

我们通过使用自己的handle1和handle2来指定两个处理器，http.Handle()函数可以调用DefaultServeMux.Handle()方法来处理请求。

```go
 func Handle(pattern string, handler Handler) { DefaultServeMux.Handle(pattern, handler) }
```

服务器的每个请求都会调用对应的ServeHTTP方法。该方法在net/http包中定义如下：

```go
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
	handler := sh.srv.Handler
	if handler == nil {
		handler = DefaultServeMux
	}
	if req.RequestURI == "*" && req.Method == "OPTIONS" {
		handler = globalOptionsHandler{}
	}
	//...
	handler.ServeHTTP(rw, req)
}
```

你也看到了当`handle == nil`处理器就对应的是DefaultServeMux。所以我们要设置之前的`Handle ：nil；`

在ServeMux对象的ServeHTTP()方法中，根据URL查找我们注册的服务器然后请求交给它处理。

虽然默认的多路复用器很好用，但仍然不推荐使用，因为它是一个全局变量，所有的代码都可以修改它。有些第三方库中可能与默认复用器产生冲突。所以推荐的做法是自定义。

### **自定义多路复用器**

```go
mux := http.NewServeMux()
mux.handleFunx("/",nil)
```

我来演示以下如何自定义多路复用器:

```go
package main

import (
	"fmt"
	"net/http"
)

func newservemux(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "NewServeMux")
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", newservemux)

	s := &http.Server{
		Addr:    ":8081",
		Handler: mux,
	}

	s.ListenAndServe()
}
```

我们的`Handle:mux`就使得复用器为mux，mux又为http.NewserveMux()方法

```go
func NewServeMux() *ServeMux { return new(ServeMux) }
```

可以看到的是NewServeMux实质上还是ServeMux。

### **ServeMux的路由匹配**

如果我们现在需要绑定三个URL分别为/，/happy，/bad。我们该如何做？与上面的代码类似

```go
 package main
 ​
 import (
     "fmt"
     "net/http"
 )
 ​
 func newservemux(w http.ResponseWriter, r *http.Request) {
     fmt.Fprintf(w, "NewServeMux")
 }
 ​
 func newservemuxhappy(w http.ResponseWriter, r *http.Request) {
     fmt.Fprintf(w, "Newservemuxhappy")
 }
 ​
 func newservemuxbad(w http.ResponseWriter, r *http.Request) {
     fmt.Fprintf(w, "NewServeMuxbad")
 }
 func main() {
     mux := http.NewServeMux()
     mux.HandleFunc("/", newservemux)
     mux.HandleFunc("/happy", newservemuxhappy)
     mux.HandleFunc("/bad", newservemuxbad)
     s := &http.Server{
         Addr:    ":8080",
         Handler: mux,
     }
 ​
     s.ListenAndServe()
 }
```

<img src="https://pic3.zhimg.com/v2-7a0b7f7c9a4652d1be611b8a4f4ef932\_b.jpg" data-caption="" data-size="normal" data-rawwidth="466" data-rawheight="162" class="origin\_image zh-lightbox-thumb" width="466" data-original="https://pic3.zhimg.com/v2-7a0b7f7c9a4652d1be611b8a4f4ef932\_r.jpg"/>

### **HttpRouter包简介**

ServeMux的一个缺陷是无法使用变量实现URL模式匹配。而HttpRouter可以，HttpRouter是一个高性能的第三方HTTP路由包，弥补了net/http包中的路由不足问题。

如何使用？

```go
 go get  github.com/julienschmidt/httprouter
```

httprouter的使用首先得使用New()函数，生成一个\*Router路由对象，然后使用GET()，方法去注册匹配的函数，最后再将这个参数传入http.ListenAndServe函数就可以监听服务。

```go
 package main
 ​
 import (
     "net/http"
     "github.com/julienschmidt/httprouter"
 )
 ​
 func Hello(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
     w.Write([]byte("Hello,httprouter!"))
 }
 ​
 func main() {
     router := httprouter.New()
     router.GET("/", Hello)
     http.ListenAndServe(":8080", router)
 }
```

HttpRouter包为常用的HTTP方法提供了GET(),POST()，方法都提供了定义。

更为重要的是，它为URL提供了两种匹配模式：

> /user/:pac 精准匹配 /user/pac  
> /user/\*pac 匹配所有模式 /user/hello

包的地址提供了详情：

```go
 package main
 ​
 import (
     "fmt"
     "net/http"
     "log"
 ​
     "github.com/julienschmidt/httprouter"
 )
 ​
 func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
     fmt.Fprint(w, "Welcome!\n")
 }
 ​
 func Hello(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
     fmt.Fprintf(w, "hello, %s!\n", ps.ByName("name"))
 }
 ​
 func main() {
     router := httprouter.New()
     router.GET("/", Index)
     router.GET("/hello/:name", Hello)
 ​
     log.Fatal(http.ListenAndServe(":8080", router))
 }
```

当然这里还有POST()，DELETE()函数的详情，就不一一解释了。

以上大概就带大家了解了http包的所有内容，以及httprouter包为http所作的URL模式匹配的拓展

下篇文章带大家继续探索go语言的其他部分

本文转自 <https://zhuanlan.zhihu.com/p/400469891?utm_id=0>，如有侵权，请联系删除。