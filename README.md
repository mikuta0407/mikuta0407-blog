# mikuta0407-blog for hugo

[平和に生きたい](https://blog.mikuta0407.net/) のリポジトリ。

他の情報はいつか今度書きます。たぶん。

## メモ

新しいポスト
```
hugo new content/posts/2024/yyyymmdd-hogefuga/index.md
mkdir content/posts/2024/yyyymmdd-hogefuga/img
```

縦向き写真
```
convert hoge.JPG -rotate 90 -strip hoge-port.JPG
```

ローカルサーバー
```
hugo server --bind <IPアドレス>
```

HEIC to JPG on macOS
```
for f in *.[Hh][Ee][Ii][Cc]; do [ -e "$f" ] && sips --setProperty format jpeg "$f" --out "${f%.*}.jpg"; done
```