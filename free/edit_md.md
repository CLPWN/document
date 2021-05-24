# markdown記法
markdownを使ってgithubに資料を投稿して行くと良い感じになります。
資料の整備にご活用ください。
---
## 見出し
```
# 見出し1
## 見出し2
### 見出し3
#### 見出し4
```

# 見出し1
## 見出し2
### 見出し3
#### 見出し4

## 箇条書き
```
- リスト1
    - ネスト リスト1_1
        - ネスト リスト1_1_1
        - ネスト リスト1_1_2
    - ネスト リスト1_2
- リスト2
- リスト3

1. 番号付きリスト1
    1. 番号付きリスト1_1
    1. 番号付きリスト1_2
1. 番号付きリスト2
1. 番号付きリスト3
```

- リスト1
    - ネスト リスト1_1
        - ネスト リスト1_1_1
        - ネスト リスト1_1_2
    - ネスト リスト1_2
- リスト2
- リスト3

1. 番号付きリスト1
    1. 番号付きリスト1_1
    1. 番号付きリスト1_2
1. 番号付きリスト2
1. 番号付きリスト3

## 引用
```
> 引用文
>> はこんな感じにかけます
```

> 引用文
>> はこんな感じにかけます

## ソースコード
```
インストールコマンドは `gem install hoge` です
```

インストールコマンドは `gem install hoge` です

```
```c <-言語名
int main(){
  return 0;
}
```
```


```c
int main(){
  return 0;
}
```

## 文字装飾
```
* 1つで*斜体*になり
* 2つで**強調**になる
```

* 1つで*斜体*になり
* 2つで**強調**になる

## 区切り線
```
-3つ
---
で区切れる
```

-3つ
---
で区切れる


## リンク
```
[Googleへのリンク](https://www.google.co.jp/)
```

[Googleへのリンク](https://www.google.co.jp/)

## 画像
```
![画像1](test1.png) <-同じフォルダにある画像ならそのままファイル名を書けばよい
![画像2](../lower/pwn/test2.png) <-別の階層にある画像も指定できる。ここではdocument/lower/pwn/にあるものを指定
![画像3](https://www.mozilla.org/media/img/firefox/new/desktop/screen-high-res.76867fcad223.png) <-画像のアドレスを貼ってもOK
```

![画像1](test1.png) 
![画像2](../lower/pwn/test2.png) 
![画像3](https://www.mozilla.org/media/img/firefox/new/desktop/screen-high-res.76867fcad223.png)
## 表組み
```
|header1|header2|header3|
|:--|--:|:--:|
|align left|align right|align center|
|a|b|c|
```

|header1|header2|header3|
|:--|--:|:--:|
|align left|align right|align center|
|a|b|c|

