---
title:  "make github blog"
date:   2021-03-29 11:38:06 +0900
categories: github.io
---
# π¦ make github blog
github blog



## λ§λ€κΈ°κΉμ§

(username).github.io λ‘ repository μ΄λ¦μ μ νλ€.

ruby μ€μΉ μν΄μ hombebrew μ€μΉ.

- ruby μ€μΉ

```swift
$ brew install rbenv ruby-build
$ rbenv install (version)
```

- jekyll μ€μΉ

`$ sudo gem install jekyll bundler` 

- make local repository

`$ git clone https://github.com/hyun99999/hyun99999.github.io.git`

- ν΄λ‘ λ°μ ν΄λ λ΄μμ jekyll μμ±

```swift
$ jekyll new ./
$ bundle install
```
- μμ±λ μ¬μ΄νΈ μ€ν
`$ bundle exec jekyll serve`

- κ°λ°μ© μλ² νμΈ
http://localhost:4000 μ μ μ

## νλ§μ μ©
https://github.com/topics/jekyll-theme μμ νλ§ λ€μ΄λ‘λ.

λ΄κ° κ³ λ₯Έ νλ§

https://github.com/mmistakes/minimal-mistakes

- jekyll μ€μΉν ν΄λλ‘ λ³΅μ¬

- νλ¬κ·ΈμΈ μ€μΉ

`$ bundle install`

- μλ² μ€ν
`$ bundle exec jekyll serve`

- μ μ©λ νλ§ local μλ²λ₯Ό ν΅ν΄μ νμΈ
<img src ="https://user-images.githubusercontent.com/69136340/112668639-17cba400-8ea2-11eb-98e7-a325eaad4930.png" width="600">

- λ°°ν¬νκΈ°(github μ μ¬λ¦°λ€.)
```swift
$ git add .
$ git commit -m "..."
$ git push origin master
```

- build fail
<p>
<img src="https://user-images.githubusercontent.com/69136340/112755848-8123f200-901d-11eb-9744-d2f229c24605.png" width="500">
<img src="https://user-images.githubusercontent.com/69136340/112755873-9731b280-901d-11eb-8fdc-ade4a84798bb.png" width="500">
</p>

https://github.com/mmistakes/minimal-mistakes#remote-theme-method
https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/#github-pages-method

μ°Έμ‘°ν΄μ ν΄κ²°.

<img src = "https://user-images.githubusercontent.com/69136340/112839786-89416780-90d9-11eb-8733-829b575560fc.png" width ="500">

## μ κΈ μμ±
- `_posts` λλ ν λ¦¬μ μ νμΌμ λ§λ¬.
```swift
Jekyll μ΄ μ΄ νμΌμ λΈλ‘κ·Έ ν¬μ€νΈλ‘ μΈμνκ² νλ €λ©΄ νμΌλͺμ λ€μ νμμ λ§μΆ°μΌ ν©λλ€:
YEAR-MONTH-DAY-title.MARKUP

μ¬κΈ°μ YEAR λ λ€ μλ¦¬μ μ«μ, MONTH μ DAY λ λ μλ¦¬ μ«μμ΄κ³ , νμ₯μ λΆλΆμ MARKUP μ νμΌμ μ¬μ©λ λ§ν¬λ€μ΄ ν¬λ§·μλλ€. μ¬λ°λ₯Έ ν¬μ€νΈ νμΌλͺμ μλ‘ λ€λ©΄ λ€μκ³Ό κ°μ΅λλ€
2011-12-31-new-years-eve-is-awesome.md
2012-09-12-how-to-write-a-blog.md
```

### μΆμ²
μΆμ²γ£https://jetalog.net/87?category=808871

μΆμ²γ£https://jekyllrb-ko.github.io/docs/posts/

μΆμ²γ£[choi2]https://iamcho2.github.io/2020/09/16/make-my-own-github-blog
