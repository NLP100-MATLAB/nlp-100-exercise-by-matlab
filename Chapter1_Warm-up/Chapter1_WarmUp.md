# 第1章: 準備運動


Copyright (c) 2020 NLP100-MATLAB



```matlab:Code
clear
```

# 00. 文字列の逆順
> 文字列”stressed”の文字を逆に（末尾から先頭に向かって）並べた文字列を得よ．




まずは string 型を使う場合。`reverse` メソッドが使えそうです。



```matlab:Code
str = "stressed";
reverse(str)
```


```text:Output
ans = "desserts"
```



 char 型を使った場合は各文字を要素とする行列っぽく操作できます。



```matlab:Code
str = 'stressed';
disp(fliplr(str))
```


```text:Output
desserts
```



char 型の場合、インデックスを使って逆順に並べ替えることも可。



```matlab:Code
str(end:-1:1)
```


```text:Output
ans = 'desserts'
```

  
# 01. 「パタトクカシーー」
> 「パタトクカシーー」という文字列の1,3,5,7文字目を取り出して連結した文字列を得よ．




これは string 型より char 型の方が使いやすそう。



```matlab:Code
str = 'パタトクカシーー';
disp(str([1,3,5,7]))
```


```text:Output
パトカー
```

# 02. 「パトカー」＋「タクシー」＝「パタトクカシーー」
> 「パトカー」＋「タクシー」の文字を先頭から交互に連結して文字列「パタトクカシーー」を得よ．




これも char 型の方がよさそう。同じ文字数なので、行列操作でやってみましょう。



```matlab:Code
str1 = 'パトカー';
str2 = 'タクシー';
str = [str1; str2]
```


```text:Output
str = 
    'パトカー'
    'タクシー'

```


```matlab:Code
str(:)'
```


```text:Output
ans = 'パタトクカシーー'
```



1x4 と 1x4 の行列を連結させて、2x4 の行列を作った後に、コラムメジャーで並べ替えました。


# 03. 円周率
> “Now I need a drink, alcoholic of course, after the heavy lectures involving quantum mechanics.”という文を単語に分解し，各単語の（アルファベットの）文字数を先頭から出現順に並べたリストを作成せよ．




単語に分解するのは string 型！



```matlab:Code
str = "Now I need a drink, alcoholic of course, after ..." + ...
    "the heavy lectures involving quantum mechanics";
str = split(str)
```


```text:Output
str = 15x1 string    
"Now"          
"I"            
"need"         
"a"            
"drink,"       
"alcoholic"    
"of"           
"course,"      
"after"        
"...the"       

```



文字数は strlength メソッドですね。



```matlab:Code
numchars = strlength(str)'
```


```text:Output
numchars = 1x15    
     3     1     4     1     6     9     2     7     5     6     5     8     9     7     9

```



1x15 の数値ベクトルが出てきました。ちょっと見ずらいので、文字列に変えて連結してみます。delimiter (区切り文字)に "" を指定することで間に空白が入らず文字連結。



```matlab:Code
join(string(numchars),"")
```


```text:Output
ans = "314169275658979"
```

# 04. 元素記号
> “Hi He Lied Because Boron Could Not Oxidize Fluorine. New Nations Might Also Sign Peace Security Clause. Arthur King Can.”という文を単語に分解し，1, 5, 6, 7, 8, 9, 15, 16, 19番目の単語は先頭の1文字，それ以外の単語は先頭に2文字を取り出し，取り出した文字列から単語の位置（先頭から何番目の単語か）への連想配列（辞書型もしくはマップ型）を作成せよ．




まずは string 型です。



```matlab:Code
str = "Hi He Lied Because Boron Could Not Oxidize Fluorine " + ...
    "New Nations Might Also Sign Peace Security Clause. Arthur King Can";
str = split(str);
```



分割しました。




先頭の１文字、２文字を取り出すとなると char 型の方がやり易そう。正規表現でもいけるかな？



```matlab:Code
idx1 = [1,5,6,7,8,9,15,16]';
idx2 = true(length(str),1); idx2(idx1) = false; idx2 = find(idx2);
% this is equivalent to
% idx2 = [2,3,4,10,11,12,13,14,17,18,19,20];
matchStr1 = regexp(str(idx1),'([a-zA-Z]{1}).*','tokens'); % 最初の1文字
matchStr2 = regexp(str(idx2),'([a-zA-Z]{2}).*','tokens'); % 最初の2文字
```



`matchStr1`、`matchStr2` ともにセル配列になっているので要注意。 




連想配列（辞書型もしくはマップ型）とのことなので、containers.Map を使ってみます。



```matlab:Code
keySet = [string(matchStr1); string(matchStr2)];
valueSet = [idx1; idx2];
M = containers.Map(keySet,valueSet);
M('Be')
```


```text:Output
ans = 4
```



Be は4番目。`table` 型の方が見やすいかな？



```matlab:Code
t = table(valueSet,keySet);
sortrows(t,'valueSet') % 出現順にソート
```

| |valueSet|keySet|
|:--:|:--:|:--:|
|1|1|"H"|
|2|2|"He"|
|3|3|"Li"|
|4|4|"Be"|
|5|5|"B"|
|6|6|"C"|
|7|7|"N"|
|8|8|"O"|
|9|9|"F"|
|10|10|"Ne"|
|11|11|"Na"|
|12|12|"Mi"|
|13|13|"Al"|
|14|14|"Si"|

# 05. n-gram
> 与えられたシーケンス（文字列やリストなど）からn-gramを作る関数を作成せよ．この関数を用い，”I am an NLPer”という文から単語bi-gram，文字bi-gramを得よ．




単語 bi-gram とは文章内で連続する単語2つ、文字 bi-gram とは各単語内での連続する文字2つと理解します。Text Analytics Toolbox だと単語 n-gram はこんな感じ。



```matlab:Code
if license('checkout','Text_Analytics_Toolbox')
    doc = tokenizedDocument("I am an NLPer"); % トークン化
    bag = bagOfNgrams(doc,'NgramLengths',2);
    bag.Ngrams
end
```


```text:Output
ans = 3x2 string    
"I"          "am"         
"am"         "an"         
"an"         "NLPer"      

```



文字 bi-gram 作ってみます。こんな関数を作ってみました。char ベクトルを入れたら、文字 n-gram を作って string 型配列で返す。文字数が少ないとそのまま返します。



```matlab:Code(Display)
function ngram = n_gram(word,NgramLength)
% word は char ベクトルを想定

N = length(word); % 文字の長さ

if N <= NgramLength
    ngram = string(word);
else
    ngram = strings(N-NgramLength,1);
    for ii = 1:N-NgramLength+1
        ngram(ii) = string(word(ii:ii+NgramLength-1));
    end
    ngram = unique(ngram); % 重複はいらない。
end

end
```



これを、`cellfun` を使って適用してみると・・ 



```matlab:Code
str = "I am an NLPer";
str = split(str);
str = cellstr(str);
cellfun(@(x) n_gram(x,2), str, 'UniformOutput', false)
```

| |1|
|:--:|:--:|
|1|'I'|
|2|'am'|
|3|'an'|
|4|4x1 string|



4 つ目だけ見難いですが、



```matlab:Code
ans{4}
```


```text:Output
ans = 4x1 string    
"LP"         
"NL"         
"Pe"         
"er"         

```

# 06. 集合
> “paraparaparadise”と”paragraph”に含まれる文字bi-gramの集合を，それぞれ, XとYとして求め，XとYの和集合，積集合，差集合を求めよ．さらに，’se’というbi-gramがXおよびYに含まれるかどうかを調べよ．


  

```matlab:Code
X = n_gram('paraparaparadise',2)
```


```text:Output
X = 8x1 string    
"ad"         
"ap"         
"ar"         
"di"         
"is"         
"pa"         
"ra"         
"se"         

```


```matlab:Code
Y = n_gram('paragraph',2)
```


```text:Output
Y = 7x1 string    
"ag"         
"ap"         
"ar"         
"gr"         
"pa"         
"ph"         
"ra"         

```



bi-gram 取れていますね。集合演算は [https://jp.mathworks.com/help/matlab/set-operations.html](https://jp.mathworks.com/help/matlab/set-operations.html) に詳細がありますが、和集合（`union`）、積集合（`intersect`）そして差集合（`setdiff`）を使います。



```matlab:Code
union(X,Y)'
```


```text:Output
ans = 1x11 string    
"ad"         "ag"         "ap"         "ar"         "di"         "gr"         "is"         "pa"         "ph"         "ra"         "se"         

```


```matlab:Code
intersect(X,Y)'
```


```text:Output
ans = 1x4 string    
"ap"         "ar"         "pa"         "ra"         

```


```matlab:Code
setdiff(X,Y)'
```


```text:Output
ans = 1x4 string    
"ad"         "di"         "is"         "se"         

```



特定の文字が含まれているかどうかは `contains` も使えそうですが、ここは単純に `=` でやります。




どれか1つでも "se" と一致しますか？ という論理変数を `any` で求めます。



```matlab:Code
any(X == "se") | any(Y == "se")
```


```text:Output
ans = 
   1

```



 'se' は存在していますね。


# 07. テンプレートによる文生成
> 引数x, y, zを受け取り「x時のyはz」という文字列を返す関数を実装せよ．さらに，x=12, y=”気温”, z=22.4として，実行結果を確認せよ．



```matlab:Code(Display)
function sentence = fromTemplate(x,y,z)

sentence = x + "時の" + y + "は" + z;

end
```



引数のデータ型として何が使われるか多少不安ではありますが・・・



```matlab:Code
fromTemplate(12,"気温",22.4)
```


```text:Output
ans = "12時の気温は22.4"
```

# 08. 暗号文
> 与えられた文字列の各文字を，以下の仕様で変換する関数cipherを実装せよ．



   -  英小文字ならば(219 - 文字コード)の文字に置換 
   -  その他の文字はそのまま出力 

> この関数を用い，英語のメッセージを暗号化・復号化せよ．




まずは char 型から始めてみます。`isstrprop` で小文字がどこにあるかを調べられるので使ってみます。



```matlab:Code
str = 'We are the borg. Resistance is futile.';
idxLower = isstrprop(str,'lower');
```



小文字だけ (219 - 文字コード) に対応する文字に変換します。`string` 型に使える文字を置き換える `replace` を試します。




まずは置換する文字を見つけてきて、`double` に与えると・・文字コードになりますので、219 - 文字コードを計算して文字に戻してみます。



```matlab:Code
unique(str(idxLower))
```


```text:Output
ans = 'abcefghilnorstu'
```


```matlab:Code
219-double(ans)
```


```text:Output
ans = 1x15    
   122   121   120   118   117   116   115   114   111   109   108   105   104   103   102

```



`char` に与えると元に戻ります。



```matlab:Code
char(ans)
```


```text:Output
ans = 'zyxvutsromlihgf'
```



上の方法を使って置換してみると、こんな感じ。



```matlab:Code
str = 'We are the borg. Resistance is futile.';
idxLower = isstrprop(str,'lower');
str(idxLower) = char(219-double(str(idxLower)))
```


```text:Output
str = 'Wv ziv gsv ylit. Rvhrhgzmxv rh ufgrov.'
```



同じ処理で元に戻りますので、これを `cipher` として実装すればOK.



```matlab:Code
idxLower = isstrprop(str,'lower');
str(idxLower) = char(219-double(str(idxLower)))
```


```text:Output
str = 'We are the borg. Resistance is futile.'
```

# 09. Typoglycemia
> スペースで区切られた単語列に対して，各単語の先頭と末尾の文字は残し，それ以外の文字の順序をランダムに並び替えるプログラムを作成せよ．ただし，長さが４以下の単語は並び替えないこととする．適当な英語の文（例えば”I couldn’t believe that I could actually understand what I was reading : the phenomenal power of the human mind .”）を与え，その実行結果を確認せよ．




`char` 型の方がやり易いかな。



```matlab:Code
sentence = ['I couldn''t believe that I could actually understand ' ...
    'what I was reading : the phenomenal power of the human mind.'];
words = split(sentence) % 各単語が１つずつセルに入ります。
```


```text:Output
words = 20x1 cell    
'I'             
'couldn't'      
'believe'       
'that'          
'I'             
'could'         
'actually'      
'understand'    
'what'          
'I'             

```



それぞれの単語について、5文字以上であれば間の文字をランダムに入れかえる処理をします。




こんな関数を作ってみました。各単語の入力は `char` 型とします。



```matlab:Code(Display)
function word2 = randomizeWord(word1)

if length(word1) <= 4
    word2 = word1;
else
    
    word2 = word1;
    tmp = word2(2:end-1);
    idx = randperm(length(tmp)); % tmp の文字数
    word2(2:end-1) = tmp(idx);
    
end
end
```



`randperm` については `tmp` の文字数を N とすると、1 から N までの数字をランダムに入れ替えた数列を返します。その数値をインデックスとすれば入れ替え可能！




ここでは `cellfun` を使ってセル内にある各単語にそれぞれ適用します。



```matlab:Code
tmp = cellfun(@randomizeWord, words, 'UniformOutput', false);
join(tmp) % 単語をつなぐ。
```


```text:Output
ans = 
    {'I cdulon't beileve that I cluod atluacly utnsnraded what I was redinag : the pnahneomel pewor of the hmuan midn.'}

```



できました。


# Appendix: 関数
## 05. n-gram

```matlab:Code
    function ngram = n_gram(word,NgramLength)
        % word は char ベクトルを想定
        N = length(word); % 文字の長さ
        
        if N <= NgramLength
            ngram = word;
        else
            ngram = strings(N-NgramLength,1);
            for ii = 1:N-NgramLength+1
                ngram(ii) = word(ii:ii+NgramLength-1);
            end
            ngram = unique(ngram); % 重複はいらない。
        end
        
    end
```

## 07. テンプレートによる文生成

```matlab:Code
    function sentence = fromTemplate(x,y,z)
        
        sentence = x + "時の" + y + "は" + z;
        
    end
```

## 09. Typoglycemia

```matlab:Code
function word2 = randomizeWord(word1)

if length(word1) <= 4
    word2 = word1;
else
    
    word2 = word1;
    tmp = word2(2:end-1);
    idx = randperm(length(tmp));
    word2(2:end-1) = tmp(idx);
    
end
end
```

