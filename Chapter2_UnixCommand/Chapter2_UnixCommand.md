# 第2章: Unix コマンド


Copyright (c) 2020 NLP100-MATLAB


> [popular-names.txt](https://nlp100.github.io/data/popular-names.txt)は，アメリカで生まれた赤ちゃんの「名前」「性別」「人数」「年」をタブ区切り形式で格納したファイルである．以下の処理を行うプログラムを作成し，[popular-names.txt](https://nlp100.github.io/data/popular-names.txt)を入力ファイルとして実行せよ．さらに，同様の処理をUNIXコマンドでも実行し，プログラムの実行結果を確認せよ．



```matlab:Code
clear
```

## 実行環境 


MATLAB R2020a (Windows 10)




popular-names.txt は本スクリプトと同じフォルダに保存されているとします。また Unix コマンドの課題ではありますが、よい解説付きの投稿が複数見られるので、MATLAB での処理に注力しています。（大変そうだからではないです）



   -  [言語処理100本ノック2020年版を解いてみた【第2章：UNIXコマンド 10〜14】](https://qiita.com/Soh1121/items/5d35e6718d3f17a9976e) 
   -  [言語処理100本ノック2020年版を解いてみた【第2章：UNIXコマンド 15〜19】](https://qiita.com/Soh1121/items/c372616f2445e98e368d) 

  
# 10. 行数のカウント
> 行数をカウントせよ．確認にはwcコマンドを用いよ．




`readtable` 関数で読み込むのが手っ取り早い。念のためヘッダー情報は変数名に使用しない設定にしておく。



```matlab:Code
tabletxt = readtable('popular-names.txt','ReadVariableNames',false);
height(tabletxt)
```


```text:Output
ans = 2780
```



2780 行ですね。もう少し low-level な関数だと `fileread` 関数でも行ける？全部読み込んで、`splitlines` 関数で改行箇所で分割します。



```matlab:Code
tmp = fileread('popular-names.txt');
txt = splitlines(tmp);
length(txt)
```


```text:Output
ans = 2781
```



一行多い・・これは最後に空の文字列が入ってしまうため。



```matlab:Code
txt(end)
```


```text:Output
ans = 
    {0x0 char}

```



Unix コマンドを使えるようにする設定準備するのがあれだったので、Windows OSのコマンドプロンプト上でできるもので代用すると・・



```matlab:Code
!find /v /c "" < popular-names.txt
```


```text:Output
2780 
```

# 11. タブをスペースに置換
> タブ1文字につきスペース1文字に置換せよ．確認にはsedコマンド，trコマンド，もしくはexpandコマンドを用いよ．




`string` 型にして `replace` 関数を使うのが簡単そう。



```matlab:Code
tmp = fileread('popular-names.txt');
txt = string(splitlines(tmp));
newtxt = replace(txt,"\t"," ");
newtxt(1:2)
```


```text:Output
ans = 2x1 string    
"Mary→F→7065→1880"    
"Anna→F→2604→1880"    

```



あれ、タブ？が残っている・・・。なんだこの矢印は。文字コード確認してみます。




まず char 型にして5文字目。



```matlab:Code
tmp = char(txt(1));
char5 = tmp(5)
```


```text:Output
char5 = '	'
```


```matlab:Code
double(char5)
```


```text:Output
ans = 9
```



文字コード 9 はタブですね。`char(9) `を `string` 型にして使ってみます。



```matlab:Code
newtxt = replace(txt,string(char(9))," ");	
newtxt(1:2)
```


```text:Output
ans = 2x1 string    
"Mary F 7065 1880"    
"Anna F 2604 1880"    

```



今度はうまくいきました。タブを他からコピペしてもうまくいきますが、MATLAB 上でタブキーを押すとスペース4つ分（自分の環境では）がタイプされるのでうまくいきません・・。



```matlab:Code
newtxt = replace(txt,"	"," ");
newtxt(1:2)
```


```text:Output
ans = 2x1 string    
"Mary F 7065 1880"    
"Anna F 2604 1880"    

```

  


要調査項目。


# 12. 1列目をcol1.txtに，2列目をcol2.txtに保存
> 各行の1列目だけを抜き出したものをcol1.txtに，2列目だけを抜き出したものをcol2.txtとしてファイルに保存せよ．確認にはcutコマンドを用いよ．




`table` 型で読み込めば（`readtable`）すぐですが、あえて `fileread` 関数でやります。



```matlab:Code
tmp = fileread('popular-names.txt');
txt = string(splitlines(tmp));
```



最後に残る空文字列を決して、`split` 関数で分割する。



```matlab:Code
txt(strlength(txt)==0) = [];
newtxt = split(txt);
newtxt(1:2,:)
```


```text:Output
ans = 2x4 string    
"Mary"       "F"          "7065"       "1880"       
"Anna"       "F"          "2604"       "1880"       

```



各文字が要素として入った 2780 x 4 の string 配列となります。




1 列目を col1.txt に、2 列目を col2.txt に保存します。



```matlab:Code
writematrix(newtxt(:,1), 'col1.txt');
writematrix(newtxt(:,2), 'col2.txt');
```

# 13. col1.txtとcol2.txtをマージ
> 12で作ったcol1.txtとcol2.txtを結合し，元のファイルの1列目と2列目をタブ区切りで並べたテキストファイルを作成せよ．確認にはpasteコマンドを用いよ．




`readcell` 関数で読み込んで、`string` 型に変換。



```matlab:Code
col1 = string(readcell('col1.txt'));
col2 = string(readcell('col2.txt'));
```



結合は `join` 関数で。デフォルトでスペース１つ空けて結合されるので、ここはタブを指定します。`"\t" `が使えないのは要調査。



```matlab:Code
col12 = join([col1,col2],char(9));
writematrix(col12, 'col12.txt');
```

# 14. 先頭からN行を出力
> 自然数Nをコマンドライン引数などの手段で受け取り，入力のうち先頭のN行だけを表示せよ．確認にはheadコマンドを用いよ．



```matlab:Code
prompt = 'What is your N value? ';
N = input(prompt);
if isnumeric(N) && N == round(N) && N > 0
    tmp = fileread('popular-names.txt');
    txt = string(splitlines(tmp));
    txt(1:N)
else
    disp('N must be a natural number.')
end
```


```text:Output
ans = 5x1 string    
"Mary→F→7065→1880"         
"Anna→F→2604→1880"         
"Emma→F→2003→1880"         
"Elizabeth→F→1939→1880"    
"Minnie→F→1746→1880"       

```



（出力はコマンド入力で 5 を渡した場合）


# 15. 末尾のN行を出力
> 自然数Nをコマンドライン引数などの手段で受け取り，入力のうち末尾のN行だけを表示せよ．確認にはtailコマンドを用いよ．



```matlab:Code
prompt = 'What is your N value? ';
N = input(prompt);
if isnumeric(N) && N == round(N) && N > 0
    tmp = fileread('popular-names.txt');
    txt = string(splitlines(tmp));
    txt(end-N-1:end-1)
else
    disp('N must be a natural number.')
end
```


```text:Output
ans = 6x1 string    
"Oliver→M→13389→2018"      
"Benjamin→M→13381→2018"    
"Elijah→M→12886→2018"      
"Lucas→M→12585→2018"       
"Mason→M→12435→2018"       
"Logan→M→12352→2018"       

```



出力はコマンド入力で 5 を渡した場合。最後の空文字の取り扱いが苦しい・・。


# 16. ファイルをN分割する
> 自然数Nをコマンドライン引数などの手段で受け取り，入力のファイルを行単位でN分割せよ．同様の処理をsplitコマンドで実現せよ．




N 分割・・。割り切れない分は最後で吸収します。



```matlab:Code
prompt = 'What is your N value? ';
N = input(prompt);
if isnumeric(N) && N == round(N) && N > 0 && N < 2780
    tmp = fileread('popular-names.txt');
    txt = string(splitlines(tmp));
    
    Ndata = ceil(2780/N); % number of rows on each file (except for the last)
    
    idx = reshape(1:(Nfiles-1)*Ndata, Ndata, []);
    for ii=1:N-1
        writematrix(txt(idx(:,ii)), "data"+ii+".txt");
    end
    writematrix(txt(idx(end)+1:end), "data"+N+".txt");
    
else
    disp('N must be a natural number and smaller than 2780.')
end
```



上の `idx` は各ファイルに収まる行番号が入るようになっています。



   -  1列目：1つ目のファイルに収まる行数（N = 100 であれば 1 から 100） 
   -  2列目：2つ目のファイルに収まる行数（N = 100 であれば 101 から 200） 
   -  3列目：3つ目のファイルに収まる行数（N = 100 であれば 201 から 300） 



といった具合。その行番号が入った配列で元データの中身を参照しながら各ファイルへ出力しています。


# 17. １列目の文字列の異なり
> 1列目の文字列の種類（異なる文字列の集合）を求めよ．確認にはcut, sort, uniqコマンドを用いよ．




そろそろ MATLAB らしく処理していいよね・・。



```matlab:Code
tabletxt = readtable('popular-names.txt','ReadVariableNames',false);
```



せっかくなので変数名を付けておきましょう。日本語文字が使えるのは R2019b から。



```matlab:Code
tabletxt.Properties.VariableNames = ["名前","性別","人数","年"];
uniqueNames = unique(tabletxt(:,1));
head(uniqueNames)
```

| |名前|
|:--:|:--:|
|1|'Abigail'|
|2|'Aiden'|
|3|'Alexander'|
|4|'Alexis'|
|5|'Alice'|
|6|'Amanda'|
|7|'Amelia'|
|8|'Amy'|

  
# 18. 各行を3コラム目の数値の降順にソート
> 各行を3コラム目の数値の逆順で整列せよ（注意: 各行の内容は変更せずに並び替えよ）．確認にはsortコマンドを用いよ（この問題はコマンドで実行した時の結果と合わなくてもよい）．




3コラム目ということは「人数」順ですね。逆順ということは多い順（descent）かな？



```matlab:Code
tabletxt = readtable('popular-names.txt','ReadVariableNames',false);
tabletxt.Properties.VariableNames = ["名前","性別","人数","年"];
```



`sort` 関数で並べ替えた順番を表すインデックス `idx` を使います。 



```matlab:Code
[~,idx] = sort(tabletxt{:,3},'descend');
tabletxt = tabletxt(idx,:);
head(tabletxt)
```

| |名前|性別|人数|年|
|:--:|:--:|:--:|:--:|:--:|
|1|'Linda'|'F'|99689|1947|
|2|'Linda'|'F'|96211|1948|
|3|'James'|'M'|94757|1947|
|4|'Michael'|'M'|92704|1957|
|5|'Robert'|'M'|91640|1947|
|6|'Linda'|'F'|91016|1949|
|7|'Michael'|'M'|90656|1956|
|8|'Michael'|'M'|90517|1958|

# 19. 各行の1コラム目の文字列の出現頻度を求め，出現頻度の高い順に並べる
> 各行の1列目の文字列の出現頻度を求め，その高い順に並べて表示せよ．確認にはcut, uniq, sortコマンドを用いよ．




ここは素直に `histcounts` 関数を使う。ただ数値かカテゴリ型でないとだめな点に注意。



```matlab:Code
tabletxt = readtable('popular-names.txt','ReadVariableNames',false);
tabletxt.Properties.VariableNames = ["名前","性別","人数","年"];

tabletxt.("名前") = categorical(tabletxt.("名前"));
[freq, names] = histcounts(tabletxt.("名前"));
```



表示のためだけですがテーブル型に整形して、さらに頻度順に整列させます。



```matlab:Code
freqNames = table(names', freq', 'VariableNames', ["名前","出現頻度"]);
freqNames = sortrows(freqNames,'出現頻度','descend');
head(freqNames)
```

| |名前|出現頻度|
|:--:|:--:|:--:|
|1|'James'|118|
|2|'William'|111|
|3|'John'|108|
|4|'Robert'|108|
|5|'Mary'|92|
|6|'Charles'|75|
|7|'Michael'|74|
|8|'Elizabeth'|73|

