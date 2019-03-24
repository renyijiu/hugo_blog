---
title: "Go指南练习题"
date: 2019-03-24T21:14:03+08:00
lastmod: 2019-03-24T21:14:03+08:00
draft: false
keywords: ['Go', 'go-tour', '练习']
description: ""
tags: ['go']
categories: ['go']
author: "renyijiu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: true
mathjax: false
---

这是自己在学习[Go 指南](http://go-tour-zh.appspot.com/list)时其中的练习题，自己编写的代码。若有问题，欢迎指正。

<!--more-->

### 练习：循环和函数

```go
package main

import (
	"fmt"
	"math"
)

func Sqrt(x float64) float64 {
	prev := 0.0
	diff := 1e-9
	z := x

	for {
		z = z - (z*z-x)/(2*z)

		if math.Abs(z-prev) < diff {
			return z
		}
		prev = z
	}
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(math.Sqrt(2))
}
```

### 练习：slice

```go
package main

import "code.google.com/p/go-tour/pic"

func Pic(dx, dy int) [][]uint8 {
	a := make([][]uint8, dy)

	for y := range a {
		b := make([]uint8, dx)

		for x := range b {

			b[x] = uint8((x + y) / 2)
		}

		a[y] = b
	}

	return a
}

func main() {
	pic.Show(Pic)
}
```

### 练习：map
```go
package main

import (
	"code.google.com/p/go-tour/wc"
	"strings"
)

func WordCount(s string) map[string]int {
	m := make(map[string]int)
	str := strings.Fields(s)

	for _, v := range str {
		m[v] += 1
	}
	return m
}

func main() {
	wc.Test(WordCount)
}
```

### 练习：斐波纳契闭包
```go
package main

import "fmt"

// fibonacci 函数会返回一个返回 int 的函数
func fibonacci() func() int {
	x, y := 0, 1

	return func() int {
		x, y = y, x+y

		return x
	}

}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```

### 练习：Stringers

```go
package main

import "fmt"

type IPAddr [4]byte

func (v IPAddr) String() string {
	return fmt.Sprintf("%v.%v.%v.%v\n", v[0], v[1], v[2], v[3])
}

func main() {
	addrs := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for n, a := range addrs {
		fmt.Printf("%v: %v\n", n, a)
	}
}
```

### 练习：错误

```go
package main

import (
	"fmt"
	"math"
)

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
	return fmt.Sprintf("cannot Sqrt negative number: %v", float64(e))
}

func Sqrt(x float64) (float64, error) {
	if x < 0 {
		return 0, ErrNegativeSqrt(x)
	} else {

		prev := 0.0
		diff := 1e-9
		z := x

		for {
			z = z - (z*z-x)/(2*z)

			if math.Abs(z-prev) < diff {
				return z, nil
			}
			prev = z
		}
	}

}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}
```

### 练习：Reader

```go
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

// TODO: 给 MyReader 添加一个 Read([]byte) (int, error) 方法

func (r MyReader) Read(b []byte) (int, error) {
	for i := range b {
		b[i] = 'A'
	}

	return len(b), nil
}

func main() {
	reader.Validate(MyReader{})
}
```

### 练习：rot13Reader

```go
package main

import (
	"io"
	"os"
	"strings"
)

type rot13Reader struct {
	r io.Reader
}

func (self rot13Reader) Read(b []byte) (int, error) {
	length, err := self.r.Read(b)

	for i := range b {
		b[i] = rot13(b[i])
	}

	return length, err
}

func rot13(a byte) byte {
	var b byte

	switch {
	case a >= 'a' && a <= 'z':
		b = 'a'
	case a >= 'A' && a <= 'Z':
		b = 'A'
	default:
		return a
	}

	return (a-b+13)%26 + b

}

func main() {
	s := strings.NewReader("Lbh penpxrq gur pbqr!")
	r := rot13Reader{s}
	io.Copy(os.Stdout, &r)
}
```

### 练习：HTTP处理

```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

type String string

type Struct struct {
	Greeting string
	Punct    string
	Who      string
}

func (s String) ServeHTTP(
	w http.ResponseWriter,
	r *http.Request) {
	fmt.Fprint(w, s)
}

func (s *Struct) ServeHTTP(
	w http.ResponseWriter,
	r *http.Request) {
	fmt.Fprint(w, s.Greeting, s.Punct, s.Who)
}

func main() {
	// your http.Handle calls here
	http.Handle("/string", String("I'm a frayed knot."))
	http.Handle("/struct", &Struct{"Hello", ":", "Gophers!"})

	log.Fatal(http.ListenAndServe("localhost:4000", nil))
}
```

### 练习：图片

```go
package main

import (
	"code.google.com/p/go-tour/pic"
	"image"
	"image/color"
)

type Image struct {
	Width  int
	Height int
}

func (i Image) ColorModel() color.Model {
	return color.RGBAModel
}

func (i Image) Bounds() image.Rectangle {
	return image.Rect(0, 0, i.Width, i.Height)
}

func (i Image) At(x, y int) color.Color {
	return color.RGBA{uint8(x), uint8(y), uint8(255), uint8(255)}
}

func main() {
	m := Image{100, 100}
	pic.ShowImage(m)
}
```

### 练习：等价二叉树
```go
package main

import (
	"fmt"
	"golang.org/x/tour/tree"
)

type Tree struct {
	Left  *Tree
	Value int
	Right *Tree
}

func Walk(t *tree.Tree, ch chan int) {
	SendValue(t, ch)
	close(ch)
}

func SendValue(t *tree.Tree, ch chan int) {
	if t != nil {
		SendValue(t.Left, ch)
		ch <- t.Value
		SendValue(t.Right, ch)
	}
}

func IsSame(t1, t2 *tree.Tree) bool {
	ch1 := make(chan int)
	ch2 := make(chan int)

	go Walk(t1, ch1)
	go Walk(t2, ch2)

	for i := range ch1 {
		if i != <-ch2 {
			return false
		}
	}

	return true
}

func main() {
	t1 := tree.New(10)
	t2 := tree.New(10)
	t3 := tree.New(5)

	fmt.Println(IsSame(t1, t2))
	fmt.Println(IsSame(t1, t3))
}
```

### 练习：Web爬虫
```go
package main

import (
	"fmt"
	"sync"
)

type UrlCounter struct {
	list map[string]int
	mu   sync.Mutex
}

var urlCounter UrlCounter

func SetValue(url string, count int) {
	urlCounter.mu.Lock()
	defer urlCounter.mu.Unlock()

	urlCounter.list[url] += count
}

func HasRead(url string) bool {
	urlCounter.mu.Lock()
	defer urlCounter.mu.Unlock()

	_, ok := urlCounter.list[url]
	return ok
}

type Fetcher interface {
	// Fetch 返回 URL 的 body 内容，并且将在这个页面上找到的 URL 放到一个 slice 中。
	Fetch(url string) (body string, urls []string, err error)
}

// Crawl 使用 fetcher 从某个 URL 开始递归的爬取页面，直到达到最大深度。
func Crawl(url string, depth int, fetcher Fetcher) {
	if depth <= 0 {
		return
	}

	if HasRead(url) {
		return
	}
	
	SetValue(url, 1)
	body, urls, err := fetcher.Fetch(url)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("found: %s %q\n", url, body)


	ch := make(chan int)
	for _, u := range urls {
		go func(url string) {
			Crawl(url, depth-1, fetcher)
			ch <- 1
		}(u)
	}

	for range urls {
		<-ch
	}
	return
}

func main() {
	urlCounter = UrlCounter{list: make(map[string]int)}
	Crawl("http://golang.org/", 4, fetcher)
}

// fakeFetcher 是返回若干结果的 Fetcher。
type fakeFetcher map[string]*fakeResult

type fakeResult struct {
	body string
	urls []string
}

func (f fakeFetcher) Fetch(url string) (string, []string, error) {
	if res, ok := f[url]; ok {
		return res.body, res.urls, nil
	}
	return "", nil, fmt.Errorf("not found: %s", url)
}

// fetcher 是填充后的 fakeFetcher。
var fetcher = fakeFetcher{
	"http://golang.org/": &fakeResult{
		"The Go Programming Language",
		[]string{
			"http://golang.org/pkg/",
			"http://golang.org/cmd/",
		},
	},
	"http://golang.org/pkg/": &fakeResult{
		"Packages",
		[]string{
			"http://golang.org/",
			"http://golang.org/cmd/",
			"http://golang.org/pkg/fmt/",
			"http://golang.org/pkg/os/",
		},
	},
	"http://golang.org/pkg/fmt/": &fakeResult{
		"Package fmt",
		[]string{
			"http://golang.org/",
			"http://golang.org/pkg/",
		},
	},
	"http://golang.org/pkg/os/": &fakeResult{
		"Package os",
		[]string{
			"http://golang.org/",
			"http://golang.org/pkg/",
		},
	},
}
```