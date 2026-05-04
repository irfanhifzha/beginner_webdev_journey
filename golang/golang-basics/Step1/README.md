Step 1: Belajar go lang penggunaan file dasar

1. buat file step1.go dengan isi
```
// Copyright 2010 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

//go:build ignore

package main

import (
	"fmt"
	"os"
)

type Page struct {
	Title string
	Body  []byte
}

func (p *Page) save() error {
	filename := p.Title + ".txt"
	return os.WriteFile(filename, p.Body, 0600)
}

func loadPage(title string) (*Page, error) {
	filename := title + ".txt"
	body, err := os.ReadFile(filename)
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

2. Compile and run di cmd windows

Compile:
```
go build step1.go
```
Run:
```
step1.exe
```
atau bisa juga run dengan:
```
step1
```

Atau langsung compile dan run
```
go run step1.go
```

3. Lihat hasilnya! ketika di run terlihat `This is a sample Page.` dan juga terbuat file TestPage.txt