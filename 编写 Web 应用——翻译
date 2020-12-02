### 1.编写 Web 应用

#### 1.0 目录

> <a href="#1.1 文章介绍">1.1 文章介绍</a>
>
> <a href="#1.2 快速入门">1.2 快速入门</a>
>
> <a href="#1.3 数据结构">1.3 数据结构</a>
>
> <a href="#1.4 net/http 包简介">1.4 net/http 包简介</a>
>
> <a href="#1.5 使用 net/http 部署 wiki 页面">1.5 使用 net/http 部署 wiki 页面</a>
>
> <a href="#1.6 页面编辑功能">1.6 页面编辑功能</a>
>
> <a href="#1.7 html/template 包">1.7 html/template 包</a>
>
> <a href="#1.8 处理不存在的（non-existent）页面">1.8 处理不存在的（non-existent）页面</a>
>
> <a href="#1.9 保存页面">1.9 保存页面</a>
>
> <a href="#1.10 错误（error）处理">1.10 错误（error）处理</a>
>
> <a href="#1.11 模板缓存（Template caching）">1.11 模板缓存（Template caching）</a>
>
> <a href="#1.12 合法性校验">1.12 合法性校验</a>
>
> <a href="#1.13 匿名函数与闭包介绍">1.13 匿名函数与闭包介绍</a>
>
> <a href="#1.14 额外的任务">1.14 额外的任务</a>
>
> <a href="#1.15 代码一览">1.15 代码一览</a>



#### 1.1 文章介绍

在本教程中包括：

* 使用 load 和 save 方法创建数据结构
* 使用 net/http 包构建 Web 应用程序
* 使用 html/template 包处理 html 模板
* 使用 regexp 包校验用户输入
* 使用闭包

前置知识

* 有一定的编程经验
* 了解基本Web技术（HTTP、HTML）
* 一些 UNIX/DOS 命令行知识



#### 1.2 快速入门

目前，运行 Go 需要有 FreeBSD、Linux、OS X 或 Windows 系统的机器。我们将使用 $ 表示命令提示符。

安装 Go（请参见 [安装说明](http://docscn.studygolang.com/doc/install)）。

在 GOPATH 中为本教程创建一个新目录，并对其进行 cd：

``` go
$ mkdir gowiki // 创建一个 gowiki 目录
$ cd gowiki // 进入新建的目录下
```

创建一个名为wiki.go的文件，在您喜欢的编辑器中打开它，然后添加以下行：

``` go
package main

import (
	"fmt"
	"io/ioutil"
)
```

我们从 Go 标准库导入 fmt 和 ioutil 包。稍后，当我们实现其他功能时，我们将向这个导入声明添加更多的包。



#### 1.3 数据结构

让我们从定义数据结构开始。首先，*Wiki* 是一种在网络上开放且可供多人协同创作的超文本系统，它由一系列相互连接的页面组成，每个页面都有标题和正文（页面内容）。这里，我们把 Page 定义为一个结构体，用两个字段表示标题和正文：

``` go
type Page struct {
    Title string
    Body  []byte
}
```

类型 []byte 表示 “字节切片”（请参阅Slices：[用法和内部函数以了解切片的详细信息](https://blog.golang.org/slices-intro)）。因为我们将要使用的 io 库所期望的类型，所以此处 Body 元素定位为一个 []byte ，而不是字符串，如下面所见。

如上所示，Page 结构描述了页面数据如何==存储在内存==中。那，如何进行持久性存储呢？我们可以通过在 Page 上创建保存方法来解决此问题：

``` go
func (p *Page) save() error {
    filename := p.Title + ".txt"
    return ioutil.WriteFile(filename, p.Body, 0600)
}
```

如代码所示：“这是一个名为 save 的方法，它使用指向 Page 的指针 p 作为其接收器——相当于方法的作用对象。它不接收任何参数，且返回参数是一个 error 类型的值"。

此方法将把 Page（页面）的 Body（正文）保存到 .txt 文本文件中。为了简单起见，我们将使用 Title（标题）作为文件名。

save 方法返回的 error 值是 WriteFile（将字节切片写入文件的标准库函数）的返回类型。save 方法返回错误值后，应用程序在写入文件时就可以根据 error 的值来处理错误。如果没有出错， Page.save() 将返回nil （指针、接口和其他类型的零值）。

八进制整数文本 0600 作为第三个参数传递给 WriteFile，指示文件应仅对当前用户具有读写权限（有关详细信息，请参见 Unix 手册页 open(2））。

除了保存页面外，我们还希望加载页面：

``` go
func loadPage(title string) *Page {
    filename := title + ".txt"
    body, _ := ioutil.ReadFile(filename)
    return &Page{Title: title, Body: body}
}
```

loadPage 函数通过 title 参数生成文件名，将文件内容读入新的 body 中，并返回一个指针，这个指针指向 title 和 body 值构造的 Page 文本。

正如第 3 行所展示的那样，Go 语言中的函数可以返回多个值，标准库函数 ioutil.ReadFile 返回 []byte 和 error 类型的两个值。但是，此处错误没有被处理：而是==用下划线（_）符号表示的 “空白标识符” 来抛弃错误返回值==（Go 语言中没有使用的变量需要抛弃掉，否则会引发编译错误）。

但如果 ReadFile 遇到错误呢？例如，文件可能不存在。所以，我们不应忽视这些错误。下面我们修改 loadPage  函数，返回 *Page 和可能出现的错误 error：

``` go
func loadPage(title string) (*Page, error) {
    filename := title + ".txt"
    body, err := ioutil.ReadFile(filename)
    if err != nil {
        return nil, err
    }
    return &Page{Title: title, Body: body}, nil
}
```

如第 3 行所示，现在接收这个函数的第二个参数，并判断它的值：如果为 nil（空值），则表示加载页面成功；如果不为空，则调用者可以处理此错误。

此时，我们有一个简单的数据结构，并且可以保存并加载文件。接下来我们会编写一个 main 函数来测试刚才所写的内容：

``` go
func main() {
    p1 := &Page{Title: "TestPage", Body: []byte("This is a sample Page.")}
    p1.save()
    p2, _ := loadPage("TestPage")
    fmt.Println(string(p2.Body))
}
```

编译并执行此代码后，将创建一个名为 TestPage.txt 的文件，其中包含 p1 的内容。然后，文件将被读入结构体p2，并将其 Body 元素打印到控制台上。

您可以按照以下方式编译和运行程序：

``` go
$ go build wiki.go
$ ./wiki
This is a sample Page.
```

> 如果使用Windows，则必须键入"wiki"而不输入"./"才能运行该程序。

<a href="#part1">点击此处获取我们刚才写过的代码</a>



#### 1.4 net/http 包简介

下面是一个简单的 Web 服务器的完整工作示例，让我们先创建一个 web.go，并写入代码：

``` go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

如第 14 行所示，main 函数首先对 http.HandleFunc 进行调用，该调用告诉 http 包使用 handler 函数处理==所有对 web 根目录==（"/"）的请求。

然后，它调用 http 包下的 ListenAndServe函数，指定它应该在任何接口上监听 8080 端口（暂时不要关注它的第二个参数nil）。这个函数将一直阻塞，直到程序终止。

ListenAndServe 函数的出参是一个错误，并且在在发生意外错误时返回。为了记录该错误，我们用 log.Fatal 封装函数调用，当 ListenAndServe 出错时，log.Fatal 打印该错误值并终止 main 函数。

回过头来看 handle，此函数类型为 http.HandlerFunc，http.ResponseWriter 和 http.Request 是它的两个参数。

http.ResponseWriter 值组装了 HTTP 服务器的响应，通过写入该响应，我们将数据发送到 HTTP 客户端。

http.Request 是表示客户端 HTTP 请求的数据结构（注意：这里必须使用指针类型），r.URL.Path 为请求 URL 的路径。后面的 [1:] 表示 “从第一个字符到结尾创建 Path 的子切片”，目的是去掉路径名中的前导字符 “/”。

如果咱们运行此程序（运行方式同上），并在任意浏览器中访问 URL：

``` go
http://localhost:8080/hello kitty
```

此时的 r.URL.Path 值为 "/hello kitty"，所以程序将提供一个包含以下内容的 html 页面：

``` html
Hi there, I love hello kitty!
```



#### 1.5 使用 net/http 部署 wiki 页面

使用 net/http 包之前，需要先引入：

``` go
import (
	"net/http"
)
```

让我们创建一个允许用户查看 wiki 页面的处理程序 viewHandler，它将处理前缀为 "/view/" 的 URL：

``` go
func viewHandler(w http.ResponseWriter, r *http.Request) {
	title := r.URL.Path[len("/view/"):]
	page := &Page{Title:title,Body:[]byte("Hello world")}
	_ = page.save()
	p, _ := loadPage(title)
	fmt.Fprintf(w, "<h1>%s</h1><div>%s</div>", p.Title, p.Body)
}
```

同样，这里使用 _ 忽略 save 和 loadPage 函数的错误返回值。这样做是为了让代码看起来更简便，但在实际场景中我们通常不这么做。

首先，该函数从请求 URL 的路径 r.URL.Path 中提取页面标题：Path 被 [len("/view/")：] 函数重新切片，以删除请求路径的前导 "/view/" 组件。这是因为路径将始终以 “/view/” 开头，而 “/view/” 不是页面标题的一部分。

先根据 title 标题生成一个 title.txt 文件。然后如第 5 行所示：该函数加载页面数据，用一个简单的 HTML 字符串修饰我们的标题字段（没学过 HTML 的编程人员可以把这些字符串看成是给普通字符套了一层样式：此处是将标题 p.Title 变为一级标题，把正文 p.Body 放入一个 div 盒子里），然后将其写入 http.ResponseWriter 类型的变量 w 中。

要使用这个 handle 处理程序，我们重写 main 函数，使用 viewHandler 来初始化 http，以处理路径 /view/ 下的所有请求：

``` go
func main() {
    http.HandleFunc("/view/", viewHandler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

为了验证上面所写，我们编译并运行：

``` go
$ go build wiki.go
$ ./wiki
```

> 如果使用 Windows，则第 2 行键入"wiki" 而不用输入"./ " 来运行该程序

运行此 Web 服务器后，用任意浏览器访问 http://localhost:8080/view/test，应显示一个名为 “test” 的页面，该页面包含 “Hello world” 字样。

<a href="#part2">点击此处获取我们刚才写过的代码</a>



#### 1.6 页面编辑功能

如上所述，我们已经实现了一个可展示 wiki 页面的程序。接下来，我们为 wiki 页面配置一个编辑功能。首先，创建两个新的 handle 处理函数：editHandle 用于显示 "编辑页面" 表单，saveHandle 负责保存输入的数据：

``` go
func main() {
    http.HandleFunc("/view/", viewHandler)
    http.HandleFunc("/edit/", editHandler)
    http.HandleFunc("/save/", saveHandler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

函数 editHandler 加载页面（如果页面不存在，则创建一个空的 Page 结构体），并将 Page 显示到 HTML 表单中：

``` go
func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    fmt.Fprintf(w, "<h1>Editing %s</h1>"+
        "<form action=\"/save/%s\" method=\"POST\">"+
        "<textarea name=\"body\">%s</textarea><br>"+
        "<input type=\"submit\" value=\"Save\">"+
        "</form>",
        p.Title, p.Title, p.Body)
}
```

这个程序功能已经变得很齐全了，但是，硬编码实现的 HTML 页面不是很推崇，且页面美观度也达不到我们追求的标准。接下来，我们将用更好的方式去实现。



#### 1.7 html/template 包

html/template 包是 Go 标准库的一部分。可以使用 html/template 将 HTML 保存在单独的文件中，这样我们就可以在不修改基础 Go 代码的情况下更改编辑页面的布局。

首先，我们必须将 html/template 添加到导入列表中：

``` go
import (
	"html/template"
	"io/ioutil"
	"net/http"
)
```

然后创建一个包含 HTML 表单的模板文件：创建一个名为 edit.html 的新文件，并添加以下行：

``` go
<h1>Editing {{.Title}}</h1>

<form action="/save/{{.Title}}" method="POST">
<div><textarea name="body" rows="20" cols="80">{{printf "%s" .Body}}</textarea></div>
<div><input type="submit" value="Save"></div>
</form>
```

修改 editHandler 函数，使用模板，而不是之前那种硬编码的 HTML ：

``` go
func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    t, _ := template.ParseFiles("edit.html")
    t.Execute(w, p)
}
```

函数 template.ParseFiles 会读取 edit.html 的内容，并返回一个指针 *template.Template。方法 t.Execute 执行模板，将生成的 HTML 写入 http.ResponseWriter。上面代码中 .Title 和 .Body 点分标识符是指 p.Title 和 p.Body。

模板指令用双大括号括起来。printf "%s" .Body 指令是一个函数调用，与调用 fmt.Printf 相同，它输出 .Body 为字符串的形式而不是原本的字节流。html/template 包有助于确保模板操作生成安全且外观合适的 HTML。例如，它会自动转义任何大于符号（>），用 &gt；替换它，以确保用户数据不会破坏表单HTML。

由于我们现在正在使用模板，让我们为我们的 viewHandler 创建一个名为 view.html 的模板：

``` go
<h1>{{.Title}}</h1>

<p>[<a href="/edit/{{.Title}}">edit</a>]</p>

<div>{{printf "%s" .Body}}</div>
```

相应修改 viewHandler：

``` go
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, _ := loadPage(title)
    t, _ := template.ParseFiles("view.html")
    t.Execute(w, p)
}
```

请注意，两个处理程序中使用了几乎完全相同的模板代码。为了代码的复用性，我们通过将模板代码移动到一个函数中来简洁代码：

``` go
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    t, _ := template.ParseFiles(tmpl + ".html")
    t.Execute(w, p)
}
```

并修改处理程序以使用该函数：

``` go
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/view/"):]
    p, _ := loadPage(title)
    renderTemplate(w, "view", p)
}
```

``` go
func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    renderTemplate(w, "edit", p)
}
```

先把 save 处理程序的那段注释掉，我们就可以再次构建和测试这个程序了。

<a href="#part3">点击此处获取我们刚才写过的代码</a>



#### 1.8 处理不存在的（non-existent）页面

之前，我们已经将合适的程序放进了 HTML 模板中，以恰当的样式访问。但是，如果我们访问一个不存在的程序，如：http://127.0.0.1:8080/view/notexists 会怎么样呢？你将看到一个只包含 HTML 的页面。这是因为它忽略了 loadPage 的错误返回，并在没有数据的情况下继续填充模板（读取 noexists.txt 文件失败）。所以，我们应该加一个判断：如果请求的页面不存在，则应将客户端重定向到编辑页面，以便可以创建内容：

``` go
func viewHandler(w http.ResponseWriter, r *http.Request) {
	title := r.URL.Path[len("/view/"):]
	page := &Page{Title:title,Body:[]byte("this is a test page")}
	_ = page.save()
	p, err := loadPage(title)
	if err != nil {
		http.Redirect(w, r, "/edit/"+title, http.StatusFound)
		return
	}
	renderTemplate(w, "view", p)
}
```

http.Redirect 函数在 HTTP 响应中添加了 http.StatusFound (302) 的 HTTP 状态码和 Location 头。



#### 1.9 保存页面

编辑页上的表单提交之后，saveHandler 函数将做出保存处理。main 中取消注释相关行，我们开始实现handler：

``` go
func saveHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/save/"):]
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    p.save()
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```

页面标题（在 URL 中提供）和表单的唯一字段 “正文” 存储在新的 Page 中，然后调用 save() 方法将数据写入文件，然后客户端被重定向到 /view/ 页。

FormValue 返回值类型为 string，我们必须将该值转换为[]byte，然后才能将其适用于 Page 结构。我们使用 []byte(body) 来执行转换。



#### 1.10 错误（error）处理

在上面的程序中，有好几个地方的错误被忽略了，这是一种不好的做法：尤其是当错误发生时，程序会发生意外行为。所以，我们需要处理错误并向用户返回错误消息。这样，如果出了问题，服务器将完全按照我们想要的方式运行，并且可以通知用户。

首先，我们处理 renderTemplate 中的错误：

``` go
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    t, err := template.ParseFiles(tmpl + ".html")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    err = t.Execute(w, p)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
    }
}
```

我们可以从上面的程序看出，如果出现了错误，http.Error 函数会向 w 中写入指定的 HTTP 响应代码（在本例中为 “内部服务器错误”）和错误消息。加入这种处理方式，我们再来修改一下 saveHandler：

``` go
func saveHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/save/"):]
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    err := p.save()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```

p.save() 期间发生的任何错误都会报告给用户。



#### 1.11 模板缓存（Template caching）

上面的代码中有一个低效率的问题：每当呈现一个页面时，renderTemplate 都会调用 ParseFiles 函数。我们可以优化一下：在程序初始化时调用 ParseFiles，将所有模板解析到单个 *Template 中。然后我们可以使用 ExecuteTemplate 方法来渲染特定的模板。

首先，我们创建一个名为 templates 的全局变量，并用 ParseFiles 初始化它：

``` go
var templates = template.Must(template.ParseFiles("edit.html", "view.html"))
```

函数 template.Must 相当于一个包装器，当传递一个非零错误值时，会进行 panic（程序异常退出）。因为模板一旦无法加载，那么大概率会对后续的其他操作产生不良影响，所以，唯一明智的做法就是退出程序。

ParseFiles 函数接受标识模板文件的任意数量的字符串参数，并将这些文件解析为模板。如果我们向程序中添加更多的模板，我们会将它们的名字添加到 ParseFiles 调用的参数中。

然后，我们修改 renderTemplate 函数以调用模板，ExecuteTemplate 方法使用适当的模板名称：

``` go
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
    err := templates.ExecuteTemplate(w, tmpl+".html", p)
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
    }
}
```

注意，模板名称是模板文件名，因此我们必须在 tmpl 参数后面附加 “.html”。



#### 1.12 合法性校验

正如您可能已经观察到的，此程序存在严重的安全缺陷：用户可以在服务器上提供任意路径来读取/写入。为了处理这种情况，我们可以编写一个函数用正则表达式来验证标题。

首先，在导入列表中添加 regexp。然后，我们可以创建一个全局变量来存储我们的验证表达式：

``` go
var validPath = regexp.MustCompile("^/(edit|save|view)/([a-zA-Z0-9]+)$")
```

函数 regexp.MustCompile 将解析并编译正则表达式，并返回 regexp.Regexp。MustCompile 与 Compile 的区别在于，如果表达式编译失败，它就会出现 panic，而 Compile 作为第二个参数返回错误。

现在，我们编写一个函数，使用 validPath 表达式来验证路径并提取页面标题：

``` go
func getTitle(w http.ResponseWriter, r *http.Request) (string, error) {
    m := validPath.FindStringSubmatch(r.URL.Path)
    if m == nil {
        http.NotFound(w, r)
        return "", errors.New("invalid Page Title")
    }
    return m[2], nil // The title is the second subexpression.
}
```

如果标题有效，则返回该标题，同时返回一个 nil 错误值。如果标题无效，该函数将向 HTTP 连接写入 “404 Not Found” 错误，并向处理程序返回错误。注意：要创建新的错误，我们必须导入errors 包。

让我们调用每个处理程序中的 getTitle：

``` go
func viewHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    p, err := loadPage(title)
    if err != nil {
        http.Redirect(w, r, "/edit/"+title, http.StatusFound)
        return
    }
    renderTemplate(w, "view", p)
}
```

``` go
func editHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    renderTemplate(w, "edit", p)
}
```

``` go
func saveHandler(w http.ResponseWriter, r *http.Request) {
    title, err := getTitle(w, r)
    if err != nil {
        return
    }
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    err = p.save()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```



#### 1.13 匿名函数与闭包介绍

这时，或许有人已经发现了，Go 语言在捕获每个处理程序中的错误条件时，会引入许多重复的代码。如果我们可以将每个处理程序包装在一个函数中，该函数执行验证和错误检查，会怎么样呢？Go 的匿名函数提供了功能抽象的强大方法，可以帮到我们。

首先，我们重写每个处理程序的函数定义以接受标题字符串：

``` go
func viewHandler(w http.ResponseWriter, r *http.Request, title string)
func editHandler(w http.ResponseWriter, r *http.Request, title string)
func saveHandler(w http.ResponseWriter, r *http.Request, title string)
```

现在我们定义一个包装函数，它接受上述类型的函数，并返回 http.HandlerFunc 类型的函数（适合传递给 http.HandleFunc 函数）：

``` go
func makeHandler(fn func (http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		// Here we will extract the page title from the Request,
		// and call the provided handler 'fn'
	}
}
```

返回的函数称为闭包，因为它包含在外部定义的变量值。在这种情况下，变量 fn（makeHandler 的单个参数）被闭包包围。变量 fn 将传入我们刚才写过的 saveHandler、editHandler或 viewHandler 。

现在，我们可以从 getTitle 获取代码，并稍作修改，然后在这里使用它：

``` go
func makeHandler(fn func(http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        m := validPath.FindStringSubmatch(r.URL.Path)
        if m == nil {
            http.NotFound(w, r)
            return
        }
        fn(w, r, m[2])
    }
}
```

makeHandler 返回的闭包是一个函数，它的入参是一个 http.ResponseWriter 和 http.Request（即 http.HandlerFunc）。它先从请求路径中提取标题，并用 validPath regexp 进行校验标题的合法性。如果标题无效，则使用 http.NotFound 函数将错误写入 ResponseWriter。如果标题有效，那么将使用 ResponseWriter、Request 和 title 作为参数来调用包含的处理器函数 fn。

现在，我们可以用 makeHandler 封装处理函数，然后才能将它们注册到 http 包中：

``` go
func main() {
    http.HandleFunc("/view/", makeHandler(viewHandler))
    http.HandleFunc("/edit/", makeHandler(editHandler))
    http.HandleFunc("/save/", makeHandler(saveHandler))

    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

最后，我们从处理程序函数中删除了对 getTitle 的调用，使它们更加简单：

``` go
func viewHandler(w http.ResponseWriter, r *http.Request, title string) {
    p, err := loadPage(title)
    if err != nil {
        http.Redirect(w, r, "/edit/"+title, http.StatusFound)
        return
    }
    renderTemplate(w, "view", p)
}
```

``` go
func editHandler(w http.ResponseWriter, r *http.Request, title string) {
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    renderTemplate(w, "edit", p)
}
```

``` go
func saveHandler(w http.ResponseWriter, r *http.Request, title string) {
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    err := p.save()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```

重新编译代码，运行应用：

``` go
$ go build wiki.go
$ ./wiki
```

访问 http://localhost:8080/view/ANewPage 时，应显示页面编辑表单。然后，你可以输入一些文本，单击 “保存”，然后被重定向到新创建的页面。

<a href="#final part">单击此处查看所有代码</a>



#### 1.14 额外的任务

以下是一些您可能需要自行处理的任务：

* 模板存放在 tmpl/，页面数据存放在 data/ ;
* 添加处理程序，使 Web 根目录重定向到 /view/FrontPage ;
* 通过使页面模板成为有效的 HTML 并添加一些 CSS 规则来修饰页面模板 ;
* 通过将【PageName】的实例转换为
  \<a href="/view/PageName">页面名称</a>（提示：您可以使用 regexp.ReplaceAllFunc 来完成此操作）。



#### 1.15 代码一览

##### part1

``` go
package main

import (
	"fmt"
	"io/ioutil"
)

type Page struct {
	Title string
	Body  []byte
}

func (p *Page) save() error {
	filename := p.Title + ".txt"
	return ioutil.WriteFile(filename, p.Body, 0600)
}

func loadPage(title string) (*Page, error) {
	filename := title + ".txt"
	body, err := ioutil.ReadFile(filename)
	if err != nil {
		return nil, err
	}
	return &Page{Title: title, Body: body}, nil
}

func main() {
	p1 := &Page{Title: "TestPage", Body: []byte("This is a sample Page.")}
	p1.save()
	p2, _ := loadPage("TestPage")
	fmt.Println(string(p2.Body))
}
```

<a href="#1.4 net/http 包简介">返回正文</a>



##### part2

``` go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
)

type Page struct {
	Title string
	Body  []byte
}

func (p *Page) save() error {
	filename := p.Title + ".txt"
	return ioutil.WriteFile(filename, p.Body, 0600)
}

func loadPage(title string) (*Page, error) {
	filename := title + ".txt"
	body, err := ioutil.ReadFile(filename)
	if err != nil {
		return nil, err
	}
	return &Page{Title: title, Body: body}, nil
}

func viewHandler(w http.ResponseWriter, r *http.Request) {
	title := r.URL.Path[len("/view/"):]
	page := &Page{Title:title,Body:[]byte("this is a test page")}
	_ = page.save()
	p, _ := loadPage(title)
	fmt.Fprintf(w, "<h1>%s</h1><div>%s</div>", p.Title, p.Body)
}

func main() {
	http.HandleFunc("/view/", viewHandler)
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

<a href="#1.6 页面编辑功能">返回正文</a>



##### part3

``` go
package main

import (
	"html/template"
	"io/ioutil"
	"log"
	"net/http"
)

type Page struct {
	Title string
	Body  []byte
}

func (p *Page) save() error {
	filename := p.Title + ".txt"
	return ioutil.WriteFile(filename, p.Body, 0600)
}

func loadPage(title string) (*Page, error) {
	filename := title + ".txt"
	body, err := ioutil.ReadFile(filename)
	if err != nil {
		return nil, err
	}
	return &Page{Title: title, Body: body}, nil
}

func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
	t, _ := template.ParseFiles(tmpl + ".html")
	t.Execute(w, p)
}

func viewHandler(w http.ResponseWriter, r *http.Request) {
	title := r.URL.Path[len("/view/"):]
	p, _ := loadPage(title)
	renderTemplate(w, "view", p)
}

func editHandler(w http.ResponseWriter, r *http.Request) {
	title := r.URL.Path[len("/edit/"):]
	p, err := loadPage(title)
	if err != nil {
		p = &Page{Title: title}
	}
	renderTemplate(w, "edit", p)
}

func main() {
	http.HandleFunc("/view/", viewHandler)
	http.HandleFunc("/edit/", editHandler)
	//http.HandleFunc("/save/", saveHandler)
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

<a href="#1.8 处理不存在的（non-existent）页面">返回正文</a>



##### final part

``` go
package main

import (
	"html/template"
	"io/ioutil"
	"log"
	"net/http"
	"regexp"
)

type Page struct {
	Title string
	Body  []byte
}

func (p *Page) save() error {
	filename := p.Title + ".txt"
	return ioutil.WriteFile(filename, p.Body, 0600)
}

func loadPage(title string) (*Page, error) {
	filename := title + ".txt"
	body, err := ioutil.ReadFile(filename)
	if err != nil {
		return nil, err
	}
	return &Page{Title: title, Body: body}, nil
}

func viewHandler(w http.ResponseWriter, r *http.Request, title string) {
	p, err := loadPage(title)
	if err != nil {
		http.Redirect(w, r, "/edit/"+title, http.StatusFound)
		return
	}
	renderTemplate(w, "view", p)
}

func editHandler(w http.ResponseWriter, r *http.Request, title string) {
	p, err := loadPage(title)
	if err != nil {
		p = &Page{Title: title}
	}
	renderTemplate(w, "edit", p)
}

func saveHandler(w http.ResponseWriter, r *http.Request, title string) {
	body := r.FormValue("body")
	p := &Page{Title: title, Body: []byte(body)}
	err := p.save()
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
		return
	}
	http.Redirect(w, r, "/view/"+title, http.StatusFound)
}

var templates = template.Must(template.ParseFiles("edit.html", "view.html"))

func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
	err := templates.ExecuteTemplate(w, tmpl+".html", p)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}
}

var validPath = regexp.MustCompile("^/(edit|save|view)/([a-zA-Z0-9]+)$")

func makeHandler(fn func(http.ResponseWriter, *http.Request, string)) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		m := validPath.FindStringSubmatch(r.URL.Path)
		if m == nil {
			http.NotFound(w, r)
			return
		}
		fn(w, r, m[2])
	}
}

func main() {
	http.HandleFunc("/view/", makeHandler(viewHandler))
	http.HandleFunc("/edit/", makeHandler(editHandler))
	http.HandleFunc("/save/", makeHandler(saveHandler))

	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

<a href="#1.14 额外的任务">返回正文</a>



原文链接：http://docscn.studygolang.com/doc/articles/wiki/
