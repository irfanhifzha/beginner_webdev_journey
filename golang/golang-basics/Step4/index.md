Step 4: Web routing handling edit/view dari txt file

1. Buat file step4.go dengan isi berikut
```
// Copyright 2010 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

//go:build ignore

package main

import (
	"html/template"
	"log"
	"net/http"
	"os"
	"regexp"
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

2. Buat file view.html pada folder yang sama dengan step4.go, isi dengan file berikut
```
<!DOCTYPE html>
<html>
<head>
    <title>{{.Title}}</title>
</head>
<body>

<h1>{{.Title}}</h1>

<div>
    <pre>{{printf "%s" .Body}}</pre>
</div>

<p>
    <a href="/edit/{{.Title}}">Edit this page</a>
</p>

</body>
</html>
```

3. Buat file edit.html pada folder yang sama dengan step4.go, isi dengan file berikut
```
<!DOCTYPE html>
<html>
<head>
    <title>Edit {{.Title}}</title>
</head>
<body>

<h1>Editing {{.Title}}</h1>

<form action="/save/{{.Title}}" method="POST">
    <div>
        <textarea name="body" rows="20" cols="80">{{printf "%s" .Body}}</textarea>
    </div>
    <div>
        <input type="submit" value="Save">
    </div>
</form>

<p>
    <a href="/view/{{.Title}}">Back to view</a>
</p>

</body>
</html>
```

3. Buat file test.txt pada folder yang sama dengan step4.go, isi bebas

4. Perhantikan hasil pada url `http://localhost:8080/view/test`

5. Coba apabila ke url yang file txt yang belum ada `http://localhost:8080/edit/belumadafile`

6. Perhatikan apa yang terjadi.

Ini adalah suatu proses routing handling, ketika ke url yang belum ada suatu file-nya, maka path akan pindah ke path `localhost/edit`, dan bisa membuat file txt baru disana. Sedangkan ketika file txt sudah ada maka bisa mengakses url seperti `localhost/view/test` bisa mengakses endpoint `test` karena sudah ada file `test.txt`  