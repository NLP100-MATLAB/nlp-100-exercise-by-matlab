# 第3章: 正規表現


Copyright (c) 2020 NLP100-MATLAB




Wikipediaの記事を以下のフォーマットで書き出したファイル [jawiki-country.json.gz](https://nlp100.github.io/data/jawiki-country.json.gz) がある．



   -  1行に1記事の情報がJSON形式で格納される 
   -  各行には記事名が”title”キーに，記事本文が”text”キーの辞書オブジェクトに格納され，そのオブジェクトがJSON形式で書き出される 
   -  ファイル全体はgzipで圧縮される 



以下の処理を行うプログラムを作成せよ．



```matlab:Code
clear
```

## 実行環境 


MATLAB R2020a (Windows 10)




使う関数は主に [regexp](https://jp.mathworks.com/help/matlab/ref/regexp.html) 関数と [regexprep](https://jp.mathworks.com/help/matlab/ref/regexprep.html) 関数！参考: [正規表現](https://www.mathworks.com/help/matlab/matlab_prog/regular-expressions.html)（MATLAB 公式ページ）


  
# 20. JSONデータの読み込み
> Wikipedia記事のJSONファイルを読み込み，「イギリス」に関する記事本文を表示せよ．問題21-29では，ここで抽出した記事本文に対して実行せよ．


  
## データファイル準備


`gunzip` 関数に URL を渡すと jawiki-country.json が作られます。既に `gunzip` 実行済みでファイルが存在する場合と処理を分けています。



```matlab:Code
if exist('jawiki-country.json','file')
    filename = 'jawiki-country.json';
else
    filename = gunzip('https://nlp100.github.io/data/jawiki-country.json.gz');
end
```

## JSON処理


json 形式の処理は `jsondecode` 関数、情報を構造体として読み取ります。




試しに1行だけ読み込んでみると、



```matlab:Code
fid = fopen(filename);
raw = fgets(fid);
str = jsondecode(raw)
```


```text:Output
str = 
    title: 'エジプト'
     text: '{{otheruses|主に現代のエジプト・アラブ共和国|古代|古代エジプト}}↵{{基礎情報 国↵|略名 =エジプト↵|漢字書き=埃及↵|日本語国名 =エジプト・アラブ共和国↵|公式国名 ={{lang|ar|'''جمهورية مصر العربية'''}}↵|国旗画像 =Flag of Egypt.svg↵|国章画像 =[[ファイル:Coat_of_arms_of_Egypt.svg|100px|エジプトの国章]]↵|国章リンク =（[[エジプトの国章|国章]]）↵|標語 =なし↵|位置画像 =Egypt (orthographic projection).svg↵|公用語 =[[アラビア語]]↵|首都 =[[File:Flag of Cairo.svg|24px]] [[カイロ]]↵|最大都市 =カイロ↵|元首等肩書 =[[近代エジプトの国家元首の一覧|大統領]]↵|元首等氏名 =[[アブドルファッターフ・アッ＝シーシー]]↵|首相等肩書 ={{ill2|エジプトの首相|en|Prime Minister of Egypt|label=首相}}↵|首相等氏名 ={{仮リンク|ムスタファ・マドブーリー|ar|مصطفى مدبولي|en|Moustafa Madbouly}}↵|面積順位 =29↵|面積大きさ =1 E12↵|面積値 =1,010,408↵|水面積率 =0.6%↵|人口統計年 =2012↵|人口順位 =↵|人口大きさ =1 E7↵|人口値 =1億人↵|人口密度値 =76↵|GDP統計年元 =2018↵|GDP値元 =4兆4,374億<ref name="economy">IMF Data and Statistics 2020年2月3日閲覧（[https://www.imf.org/external/pubs/ft/weo/2019/02/weodata/weorept.aspx?sy=2017&ey=2024&scsm=1&ssd=1&sort=country&ds=.&br=1&c=469&s=NGDP%2CNGDPD%2CPPPGDP%2CNGDPDPC%2CPPPPC&grp=0&a=&pr.x=57&pr.y=4]）</ref>↵|GDP統計年MER =2018↵|GDP順位MER =45↵|GDP値MER =2,496億<ref name="economy" />↵|GDP統計年 =2018↵|GDP順位 =21↵|GDP値 =1兆2,954億<ref name="economy" />↵|GDP/人 =13,358<ref name="economy" />↵|建国形態 =[[独立]]<br />&nbsp;-&nbsp;日付↵|建国年月日 =[[イギリス]]より<br />[[1922年]][[2月28日]]↵|通貨 =[[エジプト・ポンド]] (£)↵|通貨コード =EGP↵|時間帯 = +2↵|夏時間 =なし↵|国歌 =[[エジプトの国歌|{{lang|ar|بلادي، بلادي، بلادي}}]]{{ar icon}}<br>''我が祖国''<br>{{center|[[file:Bilady, Bilady, Bilady.ogg]]}}↵|ISO 3166-1 = EG / EGY↵|ccTLD =[[.eg]]↵|国際電話番号 =20↵|注記 =↵}}↵'''エジプト・アラブ共和国'''（エジプト・アラブきょうわこく、{{lang-ar|جمهورية مصر العربية}}）、通称'''エジプト'''は、[[中東]]（[[アラブ世界]]）および[[北アフリカ]]にある[[共和国]]。[[首都]]は[[カイロ]]。↵↵[[アフリカ大陸]]では北東端に位置し、西に[[リビア]]、南に[[スーダン]]、北東の[[シナイ半島]]では[[イスラエル]]、[[ガザ地区]]と[[国境]]を接する。北は[[地中海]]、東は[[紅海]]に面している。南北に流れる[[ナイル川]]の[[河谷]]と[[三角州|デルタ]]地帯（[[ナイル・デルタ]]）のほかは、国土の大部分の95%以上が[[砂漠]]である<ref>[https://kotobank.jp/word/エジプト-36404 エジプト]藤井宏志『日本大百科全書』小学館　2020年2月1日閲覧</ref>。ナイル河口の東に地中海と紅海を結ぶ[[スエズ運河]]がある。↵↵== 国号 ==↵正式名称は[[アラビア語]]で {{lang|ar|'''جمهورية مصر العربية'''}}（ラテン[[翻字]]: {{transl|ar|DIN|Ǧumhūrīyah Miṣr al-ʿarabīyah}}）。通称は {{lang|ar|'''مصر'''}}（[[フスハー|標準語]]: {{transl|ar|DIN|Miṣr}} ミスル、[[アラビア語エジプト方言|エジプト方言]]ほか、口語アラビア語: {{IPA|mɑsˤɾ}} マスル）。[[コプト語]]: {{Lang|cop|Ⲭⲏⲙⲓ}}（Khemi ケーミ）。↵↵アラビア語の名称'''ミスル'''は、古代から[[セム語派|セム語]]でこの地を指した名称である。なお、セム語の一派である[[ヘブライ語]]では、[[双数形]]の'''ミスライム'''（{{lang|he|מצרים}}, ミツライム）となる。↵↵公式の英語表記は '''Arab Republic of Egypt'''。通称 '''Egypt''' {{IPA-en|ˈiːdʒɨpt|}}。形容詞はEgyptian {{IPA-en|ɨˈdʒɪpʃ''ə''n|}}。エジプトの呼称は、古代[[エジプト語]]のフート・カア・プタハ（[[プタハ]]神の魂の神殿）から転じてこの地を指すようになったギリシャ語の単語である、[[ギリシャ神話]]の[[アイギュプトス (ギリシア神話)|アイギュプトス]]にちなむ。↵↵[[日本語]]の表記はエジプト・アラブ共和国。通称[[wikt:エジプト|エジプト]]。[[漢字]]では'''埃及'''と表記し、'''埃'''と略す。この[[国名の漢字表記一覧|漢字表記]]は、[[漢文]]がそのまま日本語や[[中国語]]などに輸入されたものである。英語では「イージプト」と呼ばれる。↵↵* [[1882年]] - 1922年 （{{仮リンク|イギリス領エジプト|en|History of Egypt under the British}}）↵* 1922年 - 1953年 [[エジプト王国]]↵* 1953年 - 1958年 [[エジプト共和国]]↵* 1958年 - 1971年 [[アラブ連合共和国]]↵* 1971年 - 現在 エジプト・アラブ共和国↵↵== 歴史 ==↵{{Main|エジプトの歴史}}↵↵=== 古代エジプト ===↵[[ファイル:All Gizah Pyramids.jpg|thumb|300px|right|[[ギーザ|ギザ]]の[[三大ピラミッド]]]]↵[[ファイル:Egyptiska hieroglyfer, Nordisk familjebok.png|thumb|260px|right|[[ヒエログリフ]]]]↵{{Main|古代エジプト}}↵↵「エジプトはナイルの賜物」という[[古代ギリシア]]の[[歴史家]][[ヘロドトス]]の言葉で有名なように、エジプトは豊かな[[ナイル川]]の[[三角州|デルタ]]に支えられ[[古代エジプト|古代エジプト文明]]を発展させてきた。エジプト人は[[紀元前3000年]]ごろには早くも中央集権国家を形成し、[[ピラミッド]]や[[王家の谷]]、[[ヒエログリフ]]などを通じて世界的によく知られている高度な[[文明]]を発達させた。↵↵=== アケメネス朝ペルシア ===↵3000年にわたる諸王朝の盛衰の末、[[紀元前525年]]に[[アケメネス朝]]ペルシアに支配された。↵↵=== ヘレニズム文化 ===↵[[紀元前332年]]には[[アレクサンドロス3世|アレクサンドロス大王]]に征服された。その後、[[ギリシャ人|ギリシア系]]の[[プトレマイオス朝]]が成立し、[[ヘレニズム]]文化の中心のひとつとして栄えた。↵↵=== ローマ帝国 ===↵プトレマイオス朝は[[紀元前30年]]に滅ぼされ、エジプトは[[ローマ帝国]]の[[属州]]となり[[アエギュプトゥス]]と呼ばれた。ローマ帝国の統治下では[[キリスト教]]が広まり、[[コプト教会]]が生まれた。ローマ帝国の分割後は[[東ローマ帝国]]に属し、豊かな[[穀物]]生産でその繁栄を支えた。↵↵=== イスラム王朝 ===↵7世紀に[[イスラム教|イスラム化]]。[[639年]]に[[イスラム帝国]]の[[将軍]][[アムル・イブン・アル＝アース]]によって征服され、[[ウマイヤ朝]]および[[アッバース朝]]の一部となった。アッバース朝の支配が衰えると、そのエジプト[[総督]]から自立した[[トゥールーン朝]]、[[イフシード朝]]の短い支配を経て、[[969年]]に現在の[[チュニジア]]で興った[[ファーティマ朝]]によって征服された。これ以来、[[アイユーブ朝]]、[[マムルーク朝]]とエジプトを本拠地として[[歴史的シリア|シリア地方]]まで版図に組み入れた[[イスラム王朝]]が500年以上にわたって続く。特に250年間続いたマムルーク朝の下で[[中央アジア]]や[[カフカス]]などアラブ世界の外からやってきた[[マムルーク]]（奴隷軍人）による支配体制が確立した。↵↵=== オスマン帝国 ===↵[[1517年]]に、マムルーク朝を滅ぼしてエジプトを属州とした[[オスマン帝国]]のもとでもマムルーク支配は温存された（{{仮リンク|エジプト・エヤレト|en|Egypt Eyalet}}）。↵↵=== ムハンマド・アリー朝 ===↵[[ファイル:ModernEgypt, Muhammad Ali by Auguste Couder, BAP 17996.jpg|thumb|180px|[[ムハンマド・アリー]]]]↵[[1798年]]、[[フランス]]の[[ナポレオン・ボナパルト]]による[[エジプト・シリア戦役|エジプト遠征]]をきっかけに、エジプトは[[近代国家]]形成の時代を迎える。フランス軍撤退後、混乱を収拾して権力を掌握したのはオスマン帝国が派遣した[[アルバニア人]]部隊の隊長としてエジプトにやってきた軍人、[[ムハンマド・アリー]]であった。彼は実力によってエジプト総督に就任すると、マムルークを打倒して総督による中央集権化を打ち立て、[[経済]]・[[軍事]]の近代化を進め、エジプトをオスマン帝国から半ば独立させることに成功した。アルバニア系ムハンマド・アリー家による[[世襲]]政権を打ち立てた（[[ムハンマド・アリー朝]]）。しかし、当時の世界に勢力を広げた[[ヨーロッパ]][[列強]]はエジプトの独立を認めず、また、ムハンマド・アリー朝の急速な近代化政策による社会矛盾は結局、エジプトを列強に経済的に従属させることになった。↵↵=== イギリスの進出 ===↵ムハンマド・アリーは[[綿花]]を主体とする農産物[[専売制]]をとっていたが、1838年に宗主オスマン帝国が[[イギリス]]と自由貿易協定を結んだ。ムハンマド・アリーが1845年に三角州の堰堤を着工。死後に専売制が崩壊し、また堰堤の工期も延びて3回も支配者の交代を経た1861年、ようやく一応の完工をみた。1858年末には国庫債券を発行しなければならないほどエジプト財政は窮迫していた。[[スエズ運河会社]]に払い込む出資金の不足分は、シャルル・ラフィット（[[:fr:Charles Laffitte|Charles Laffitte]]）と割引銀行（現・[[BNPパリバ]]）から借り、国庫債券で返済することにした。[[イスマーイール・パシャ]]が出資の継続を認めたとき、フランスの[[ナポレオン3世]]の裁定により契約責任を問われ、違約金が[[自転車操業]]に拍車をかけた。[[1869年]]、エジプトはフランスとともに[[スエズ運河]]を開通させた。この前後（1862 - 1873年）に8回も[[外債]]が起債され、額面も次第に巨額となっていた。エジプトはやむなくスエズ運河会社持分を398万[[スターリング・ポンド|ポンド]]でイギリスに売却したが、1876年4月に[[デフォルト]]した<ref>西谷進、「[https://doi.org/10.20624/sehs.37.2_113 一九世紀後半エジプト国家財政の行詰まりと外債 (一)]」 『社会経済史学』 1971年 37巻 2号 p.113-134,216↵, {{doi|10.20624/sehs.37.2_113}}</ref>。↵↵英仏が負債の償還をめぐって争い、エジプトの蔵相は追放された。[[イタリア]]や[[オーストリア]]も交えた負債委員会が組織された。2回目のリスケジュールでイスマーイール一族の直轄地がすべて移管されたが、土地税収が滞った<ref>西谷進、「[https://doi.org/10.20624/sehs.37.3_283 一九世紀後半エジプト国家財政の行詰まりと外債 (二)]」 社会経済史学 1971年 37巻 3号 p.283-311,330-33, {{doi|10.20624/sehs.37.3_283}}</ref>。↵↵[[1882年]]、[[アフマド・オラービー]]が中心となって起きた反英運動（[[ウラービー革命]]）がイギリスによって武力鎮圧された。エジプトはイギリスの[[保護国]]となる。結果として、政府の教育支出が大幅カットされるなどした。[[1914年]]には、[[第一次世界大戦]]によってイギリスがエジプトの名目上の[[宗主国]]であるオスマン帝国と開戦したため、エジプトはオスマン帝国の宗主権から切り離された。さらに[[サアド・ザグルール]]の逮捕・国外追放によって反英独立運動たる[[エジプト革命 (1919年)|1919年エジプト革命]]が勃発し、英国より主政の国として独立した。↵↵=== 独立・エジプト王国 ===↵第一次大戦後の[[1922年]][[2月28日]]に'''[[エジプト王国]]'''が成立し、翌年イギリスはその[[独立]]を認めたが、その後もイギリスの間接的な支配体制は続いた。↵↵エジプト王国は[[立憲君主制]]を布いて議会を設置し、緩やかな近代化を目指した。[[第二次世界大戦]]では、[[枢軸国軍]]がイタリア領リビアから侵攻したが、英軍が撃退した（[[北アフリカ戦線]]）。第二次世界大戦前後から[[パレスチナ問題]]の深刻化や、1948年から1949年の[[パレスチナ戦争]]（[[第一次中東戦争]]）での[[イスラエル]]への敗北、経済状況の悪化、[[ムスリム同胞団]]など政治のイスラム化（[[イスラム主義]]）を唱える社会勢力の台頭によって次第に動揺していった。↵↵=== エジプト共和国 ===↵この状況を受けて[[1952年]]、軍内部の秘密組織[[自由将校団]]が[[クーデター]]を起こし、国王[[ファールーク1世 (エジプト王)|ファールーク1世]]を亡命に追い込み、ムハンマド・アリー朝を打倒した（[[エジプト革命 (1952年)|エジプト革命]]<ref>片山正人『現代アフリカ・クーデター全史』叢文社 2005年 ISBN 4-7947-0523-9 p49</ref>）。生後わずか半年の[[フアード2世 (エジプト王)|フアード2世]]を即位させ、[[自由将校団]]団長の[[ムハンマド・ナギーブ]]が首相に就任して権力を掌握した。さらに翌年の1953年、国王を廃位して共和政へと移行、[[ムハンマド・ナギーブ|ナギーブ]]が首相を兼務したまま初代大統領となり、'''エジプト共和国'''が成立した。↵↵=== ナーセル政権 ===↵[[ファイル:Gamal Nasser.jpg|thumb|180px|right|[[ガマール・アブドゥル＝ナーセル]]。[[第二次中東戦争]]に勝利し、スエズ運河を国有化した。ナーセルの下でエジプトは[[汎アラブ主義]]の中心となった]]↵[[1956年]]、第2代大統領に就任した[[ガマール・アブドゥル＝ナーセル]]のもとでエジプトは[[冷戦]]下での中立外交と[[汎アラブ主義|汎アラブ主義（アラブ民族主義）]]を柱とする独自の政策を進め、[[第三世界]]・[[アラブ諸国]]の雄として台頭する。同年にエジプトは[[スエズ運河国有化宣言|スエズ運河国有化]]を断行し、これによって勃発した[[第二次中東戦争]]（スエズ戦争）で政治的に勝利を収めた。[[1958年]]には[[シリア]]と連合して'''[[アラブ連合共和国]]'''を成立させた。しかし[[1961年]]にはシリアが連合から脱退し、[[国家連合]]としてのアラブ連合共和国はわずか3年で事実上崩壊した。さらに[[1967年]]の[[第三次中東戦争]]は惨敗に終わり、これによってナーセルの権威は求心力を失った。↵↵=== サーダート政権 ===↵[[1970年]]に急死したナーセルの後任となった[[アンワル・アッ＝サーダート]]は、自ら主導した[[第四次中東戦争]]後に[[ソビエト連邦]]と対立して[[アメリカ合衆国]]など[[西側諸国]]に接近。[[社会主義]]的経済政策の転換、[[イスラエル]]との融和など、ナーセル体制の切り替えを進めた。[[1971年]]には、国家連合崩壊後もエジプトの国号として使用されてきた「アラブ連合共和国」の国号を捨てて'''エジプト・アラブ共和国'''に改称した。また、サーダートは、経済の開放などに舵を切るうえで、左派に対抗させるべくイスラーム主義勢力を一部容認した。しかしサーダートは、イスラエルとの和平を実現させたことの反発を買い、[[1981年]]に[[イスラム過激派]]の[[ジハード団]]によって[[暗殺]]された。↵↵=== ムバーラク政権 ===↵[[ファイル:Hosni Mubarak ritratto.jpg|thumb|180px|[[アラブの春]]で失脚するまで30年以上にわたり長期政権を維持した[[ホスニー・ムバーラク]]]]↵[[イラク]]の[[クウェート侵攻]]はエジプトの国際収支を悪化させた。サーダートに代わって副大統領から大統領に昇格した[[ホスニー・ムバーラク]]は、対米協調外交を進める一方、[[開発独裁]]的な[[政権]]を20年以上にわたって維持した。↵↵ムバラク政権は1990年12月に「1000日計画」と称する経済改革案を発表した。クウェート解放を目指す[[湾岸戦争]]では[[多国籍軍]]へ2万人を派兵し、これにより約130億ドルも[[対外債務]]を減らすという外交成果を得た。累積債務は500億ドル規模であった。軍事貢献により帳消しとなった債務は、クウェート、[[サウジアラビア]]に対するものと、さらに対米軍事債務67億ドルであった。1991年5月には[[国際通貨基金]]のスタンドバイクレジットおよび[[世界銀行]]の構造調整借款（SAL）が供与され、[[パリクラブ]]において200億ドルの債務削減が合意された。エジプト経済の構造調整で画期的だったのは、ドル・ペッグによる為替レート一本化であった<ref>『エジプトの経済発展の現状と課題』 [[海外経済協力基金]]開発援助研究所 1998年 23頁</ref>。↵↵海外の[[機関投資家]]に有利な条件が整えられていった。イスラム主義運動は厳しく[[弾圧]]され、[[ザカート|喜捨]]の精神は失われていった。[[1997年]]には[[イスラム集団]]による[[ルクソール事件]]が発生している。1999年にイスラム集団は武装闘争放棄を宣言し、近年、観光客を狙った事件は起こっていない。しかし、ムバーラクが大統領就任と同時に発令した[[非常事態法]]は、彼が追放されるまで30年以上にわたって継続された<ref>[http://www.cnn.co.jp/world/30001719.html エジプト副大統領が野党代表者らと会談、譲歩示す]</ref>。↵↵2002年6月、エジプト政府は15億ドルの[[ユーロ債]]を起債したが、2002年から2003年に為替差損を被り、対外債務を増加させた<ref>IMF, ''Arab Republic of Egypt: Selected Issues'', 2005, [https://books.google.co.jp/books?id=aZDSd_fjJzkC&pg=PT49&dq=reschedule+egypt+eurobond&hl=ja&sa=X&ved=0ahUKEwiD0qHpn8DbAhWLWbwKHXRrDGQQ6AEIJzAA#v=onepage&q=reschedule%20egypt%20eurobond&f=false]</ref>。↵↵=== ムルシー政権 ===↵[[ファイル:Mohamed Morsi-05-2013.jpg|thumb|180px|民主化後初の大統領だった[[ムハンマド・ムルシー]]]]↵{{Main|エジプト革命 (2011年)|2012年エジプト大統領選挙}}↵[[チュニジア]]の[[ジャスミン革命]]に端を発した近隣諸国の民主化運動がエジプトにおいても波及し、[[2011年]]1月、30年以上にわたって独裁体制を敷いてきたムバーラク大統領の辞任を求める大規模なデモが発生した。同2月には大統領支持派によるデモも発生して騒乱となり、国内主要都市において大混乱を招いた。大統領辞任を求める声は日に日に高まり、2月11日、ムバーラクは大統領を辞任し、全権が[[エジプト軍最高評議会]]に委譲された。同年12月7日には{{仮リンク|カマール・ガンズーリ|en|Kamal Ganzouri}}を暫定首相とする政権が発足した。その後、2011年12月から翌年1月にかけて人民議会選挙が、また2012年5月から6月にかけて大統領選挙が実施され[[ムハンマド・ムルシー]]が当選し、同年6月30日の大統領に就任したが、人民議会は大統領選挙決選投票直前に、選挙法が違憲との理由で裁判所から解散命令を出されており、立法権は軍最高評議会が有することとなった。↵↵2012年11月以降、新憲法の制定などをめぐって反政府デモや暴動が頻発した（{{仮リンク|2012年-13年エジプト抗議運動|en|2012–13 Egyptian protests}}）。ムルシー政権は、政権への不満が大規模な暴動に発展するにつれて、当初の警察改革を進める代わりに既存の組織を温存する方向に転換した。{{仮リンク|ムハンマド・イブラヒーム・ムスタファ|ar|محمد إبراهيم مصطفى|label=ムハンマド・イブラヒーム}}が内相に就任した[[2013年]]1月以降、治安部隊による政治家やデモ隊への攻撃が激化。1月末には当局との衝突でデモ参加者など40人以上が死亡したが、治安部隊への調査や処罰は行われていない<ref>「『アラブの春』の国で繰り返される悪夢」 エリン・カニンガム 『Newsweek[[ニューズウィーク]]日本版』 2013年3月5日号</ref>。イブラヒーム内相は、「国民が望むならば辞任する用意がある」と2月に述べている<ref>{{Cite news↵|url=http://english.ahram.org.eg/News/63894.aspx↵|title=I will leave my position if people want: Egypt's interior …'

```


```matlab:Code
fclose(fid);
```



こんな感じです。`title` と `text` が char 型で取れています。




`fgets` を繰り返し読んで一行づつ処理してもいいですが、ここでは



   -  `fileread` 関数で一気に読み込んで、 
   -  string 型にして、`splitlines` 関数で行ごとに分割 
   -  毎行 `jsondecode` で処理して `table` 化 



でやってみます。



```matlab:Code
text = string(fileread(filename));
jsontexts = splitlines(text); % 分割
jsontexts(strlength(jsontexts) == 0) = []; % 空の行を削除

% table 型変数を事前定義
T = table('Size',[length(jsontexts),2], ...
    'VariableTypes',["string","string"], ...
    'VariableNames',["title","text"]);

% 各行処理
for ii=1:length(jsontexts)
    % 構造体を table 型に変換
    tmp = struct2table(jsondecode(jsontexts(ii)));
    
    % string 型にしておくとセルを使わず結合できるので便利。
    tmp.title = string(tmp.title); % string 型に変更
    tmp.text = string(tmp.text); % string 型に変更
    
    T(ii,:) = tmp; % 連結
end
```



**メモ：**`text` と `title` は char 型で返ってくるので明示的に string 型に変更していますが、`T(ii,:) = tmp;` だけでもデータ型は自動で変更されます。`T` の変数を string 型だと事前定義しているため。



```matlab:Code
head(T)
```

| |title|text|
|:--:|:--:|:--:|
|1|"エジプト"|"{{otheruses|主に現代のエジプト・アラブ共和国|古代|古代エジプト\|



こんな感じ。


## イギリス抽出


イギリスだけ取り出せばよいので、



```matlab:Code
UKWiki = T(T.title == "イギリス",:)
```

| |title|text|
|:--:|:--:|:--:|
|1|"イギリス"|"{{redirect|UK\|

# 21. カテゴリ名を含む行を抽出
> 記事中でカテゴリ名を宣言している行を抽出せよ．


  


[マークアップ早見表](https://ja.wikipedia.org/wiki/Help:%E6%97%A9%E8%A6%8B%E8%A1%A8)（Wikipedia）によるとカテゴリ名のマークアップは



   -  [[Category:ヘルプ|はやみひよう]] 



であり、テキストを直接と確認すると以下の行が該当する模様です。Wikipedia ではページ最下部のカテゴリの欄にリンクが追加されるマークアップだそうな。**メモ：**カテゴリ名は分かるが、 | のあと（上の例だと「はやみひよう」）は何だろう・・



```matlab:Code(Display)
[[Category:イギリス|*]]\n'
[[Category:英連邦王国|*]]\n'
[[Category:G8加盟国]]\n'
[[Category:欧州連合加盟国]]\n'
[[Category:海洋国家]]\n'
[[Category:君主国]]\n'
[[Category:島国|くれいとふりてん]]\n'
[[Category:1801年に設立された州・地域]]'
```



これを見つける正規表現を考えます。使う関数は [regexp](https://jp.mathworks.com/help/matlab/ref/regexp.html) 関数。



```matlab:Code
startIndex = regexp(UKWiki.text,'\[{2}Category:[^\]]*\]{2}')'
```


```text:Output
startIndex = 9x1    
       67808
       67828
       67851
       67872
       67891
       67914
       67932
       67953
       67969

```



9つ見つかりました。それぞれの意図は・・



```matlab:Code(Display)
\[{2}     % カッコ [ が2つ（\ を前に付けて文字としての [ として識別させます）
Category: % 後に続く文字列
[^\]]*    % ^\] は \] 以外の文字列。 * は 0 回以上の繰り返し。+ にすると 1 回以上。
\]{2}     % カッコ ] が2つ
```



です。記号の場合、正規表現の演算子と区別するために \ をいちいち付ける必要がある点は注意。




実際に合致した文字列を出すにはオプションに `'match'` を加えます。



```matlab:Code
match = regexp(UKWiki.text,'\[{2}Category:[^\]]*\]{2}','match')'
```


```text:Output
match = 9x1 string    
"[[Category:イギリス|*]]"          
"[[Category:イギリス連邦加盟国]]"       
"[[Category:英連邦王国|*]]"         
"[[Category:G8加盟国]]"           
"[[Category:欧州連合加盟国|元]]"       
"[[Category:海洋国家]]"            
"[[Category:現存する君主国]]"         
"[[Category:島国]]"              
"[[Category:1801年に成立した国家・領域]]" 

```

# 22. カテゴリ名の抽出
> 記事のカテゴリ名を（行単位ではなく名前で）抽出せよ．


  


カテゴリ名だけを取り出す場合はトークン（正規表現の一部をかっこで囲んで定義した、一致テキストの一部）を取ります。




先ほど `'match'` にしていたところを `'tokens'` に変えます。そしてカテゴリ名のところをカッコで囲んでいる点に注意。



```matlab:Code
out = regexp(UKWiki.text,'\[{2}Category:([^\]]*)\]{2}','tokens')'
```

| |1|
|:--:|:--:|
|1|"イギリス|*"|
|2|"イギリス連邦加盟国"|
|3|"英連邦王国|*"|
|4|"G8加盟国"|
|5|"欧州連合加盟国|元"|
|6|"海洋国家"|
|7|"現存する君主国"|
|8|"島国"|
|9|"1801年に成立した国家・領域"|



カテゴリ名として | より前の部分だけ取り出す場合は・・



```matlab:Code(Display)
\[{2}      % カッコ [ が2つ（\ を前に付けて文字としての [ として識別させます）
Category:  % 後に続く文字列
([^\]\|]*) % ^\]\| は \] と \| 以外の文字列。カッコで囲んでトークン化。* は 0 回以上の繰り返しを意味。
\|?        % \| が無いものもあるので、0 回、または 1 回を意味する ?
[^\]\|]*   % \] と \| 以外の文字列が 0 回以上
\]{2}      % カッコ ] が2つ
```


```matlab:Code
out = regexp(UKWiki.text,'\[{2}Category:([^\]\|]*)\|?[^\]\|]*\]{2}','tokens')'
```

| |1|
|:--:|:--:|
|1|"イギリス"|
|2|"イギリス連邦加盟国"|
|3|"英連邦王国"|
|4|"G8加盟国"|
|5|"欧州連合加盟国"|
|6|"海洋国家"|
|7|"現存する君主国"|
|8|"島国"|
|9|"1801年に成立した国家・領域"|

# 23. セクション構造
> 記事中に含まれるセクション名とそのレベル（例えば”== セクション名 ==”なら1）を表示せよ．


  


[マークアップ早見表](https://ja.wikipedia.org/wiki/Help:%E6%97%A9%E8%A6%8B%E8%A1%A8)（Wikipedia）によるとセクションのマークアップは



   -  == Level 2 == 
   -  === Level 3 === 



の様です。セクション名（Leve 2 や Level 3）とそのレベル感（== の数から 1 引いた数字）を取ってきます。



```matlab:Code
regexp(UKWiki.text,'(={2,})([^=]+)\1','match')'
```


```text:Output
ans = 55x1 string    
"==国名=="          
"==歴史=="          
"==地理=="          
"===主要都市==="      
"===気候==="        
"==政治=="          
"===元首==="        
"===法==="         
"===内政==="        
"===地方行政区分==="    

```



それらしいものが検出されています。正規表現の意味は



```matlab:Code(Display)
(={2,})   % = が 2 回以上。() で囲んでトークン化。
([^=]+)   % = 以外の文字列を 1 回以上。() で囲んでトークン化。
\1        % 1つ目のトークンの内容をリピート。
```



です。




1 つ目のトークン（= の繰り返し）と 2 つ目のトークン（セクション名）を取ってきて、セクション名とそのレベルを table に纏めます。



```matlab:Code
out = regexp(UKWiki.text,'(={2,})([^=]+)\1','tokens')';
levels = cellfun(@(x) strlength(x(1)),out)-1;
sectionNames = cellfun(@(x) x(2),out);
table(sectionNames,levels)
```

| |sectionNames|levels|
|:--:|:--:|:--:|
|1|"国名"|1|
|2|"歴史"|1|
|3|"地理"|1|
|4|"主要都市"|2|
|5|"気候"|2|
|6|"政治"|1|
|7|"元首"|2|
|8|"法"|2|
|9|"内政"|2|
|10|"地方行政区分"|2|
|11|"外交・軍事"|2|
|12|"経済"|1|
|13|"鉱業"|2|
|14|"農業"|2|

# 24. ファイル参照の抽出
> 記事から参照されているメディアファイルをすべて抜き出せ．


  


またまた[マークアップ早見表](https://ja.wikipedia.org/wiki/Help:%E6%97%A9%E8%A6%8B%E8%A1%A8)（Wikipedia）によるとメディアファイルは




**[[ファイル:Wikipedia-logo-v2-ja.png|thumb|説明文]]**




とのこと。




ただ、thumb や 説明文が付いていないものもあります・・ややこしい。




例：[[ファイル:United States Navy Band - God Save the Queen.ogg]]




まずは該当箇所を検出してみます。



```matlab:Code
regexp(UKWiki.text,'\[{2}ファイル:[^\]]+\]{2}','match')'
```


```text:Output
ans = 27x1 string    
"[[ファイル:Royal Coat of Arms o…  
"[[ファイル:United States Navy B…  
"[[ファイル:Descriptio Prime Tab…  
"[[ファイル:Lenepveu, Jeanne d'A…  
"[[ファイル:London.bankofengland…  
"[[ファイル:Battle of Waterloo 1…  
"[[ファイル:Uk topo en.jpg|thumb…  
"[[ファイル:BenNevis2005.jpg|thu…  
"[[ファイル:Population density U…  
"[[ファイル:2019 Greenwich Penin…  

```



正規表現を分解すると・・



```matlab:Code(Display)
\[{2}     % [ を 2 つ
ファイル:  % ファイル:という文字列があって
[^\]]+    % ] という文字列以外の文字列が 1 回以上繰り返されて
\]{2}     % ] が 2 つ
```



に合致する箇所を検索しました。




ここからファイル名だけ取り出します。基本構文での「説明文」のところが複雑なので、1つ目の `|` が出てくるまで（もしくはそのまま終了するまで）の文字列をファイル名として取り出してみます。



```matlab:Code
regexp(UKWiki.text,'\[{2}ファイル:([^\]\|]*)(?:\]{2}|\|)','tokens')'
```

| |1|
|:--:|:--:|
|1|"Royal Coat of Arms of the United Kingdom.svg"|
|2|"United States Navy Band - God Save the Queen.ogg"|
|3|"Descriptio Prime Tabulae Europae.jpg"|
|4|"Lenepveu, Jeanne d'Arc au siège d'Orléans.jpg"|
|5|"London.bankofengland.arp.jpg"|
|6|"Battle of Waterloo 1815.PNG"|
|7|"Uk topo en.jpg"|
|8|"BenNevis2005.jpg"|
|9|"Population density UK 2011 census.png"|
|10|"2019 Greenwich Peninsula \& Canary Wharf.jpg"|
|11|"Birmingham Skyline from Edgbaston Cricket Ground crop.jpg"|
|12|"Leeds CBD at night.jpg"|
|13|"Glasgow and the Clyde from the air (geograph 4665720).jpg"|
|14|"Palace of Westminster, London - Feb 2007.jpg"|



正規表現を分解すると・・



```matlab:Code(Display)
\[{2}         % [ を 2 つ
ファイル:      % ファイル:という文字列があって
([^\]\|]*)    % ] と | の文字列以外の文字列が 0 回以上繰り返されて
(?:\}{2}|\|)  % ] が 2 つもしくは | が登場するまで。(?:exp1|exp2) は exp1 or exp2 でグループ化するがトークンは取らない
```



を探しています。ファイル名部分をトークン化している点にも注意。




確認ですが、これだと




|国章画像 = [[ファイル:Royal Coat of Arms of the United Kingdom.svg|85px|イギリスの国章]]




のファイル名までしか検出していません。最後の `]]` まで検出するなら（他によいやり方あれば教えてください・・）



```matlab:Code
regexp(UKWiki.text,'\[{2}ファイル:([^\]\|]*)(?:\]{2}|\|[^\]\|]*)+','tokens')';
```


```matlab:Code(Display)
\[{2}                  % [ を 2 つ
ファイル:               % ファイル:という文字列があって
([^\]\|]*)             % ] と | の文字列以外の文字列が 0 回以上繰り返されて
(?:\}{2}|\|)           % ] | が2つが登場するまで。(?:exp1|exp2) は exp1 or exp2 でグループ化するがトークンは取らない
(?:\]{2}|\|[^\]\|]*)+  % ] が2つ もしくは \| のあとに [^\]\|]* (] と | の文字列以外の文字列が 0 回以上繰り返され) る
                       % という条件を 1 回以上繰り返し。
                       % すなわち ] が2つ 出てくるまでを検出！
```



かなり苦しい正規表現となりました・・。


# 25. テンプレートの抽出
> 記事中に含まれる「基礎情報」テンプレートのフィールド名と値を抽出し，辞書オブジェクトとして格納せよ．


  
## まずは基礎情報部分抽出


テンプレートの例:[Template:基礎情報 国](https://ja.wikipedia.org/wiki/Template:%E5%9F%BA%E7%A4%8E%E6%83%85%E5%A0%B1_%E5%9B%BD) (Wikipedia) によると



```matlab:Code(Display)
{{基礎情報 国 
|自治領等 = 
|略名 =  
|日本語国名 =  
|公式国名 = 
（中略）
}}
```



のような構文の様です。これは「国」の基本情報ですが他にもいろいろあるんでしょうね。




では早速



```matlab:Code(Display)
\{{2}    % { を 2 つ
基礎情報  % 「基礎情報」という文字列があって
[^}]*    % } 以外の文字列が 0 個以上
\}{2}    % } 2 つで閉じられるところ。
```


```matlab:Code
out = regexp(UKWiki.text,'\{{2}基礎情報[^}]*\}{2}','match')
```


```text:Output
out = 
    "{{基礎情報 国
     |略名  =イギリス
     |日本語国名 = グレートブリテン及び北アイルランド連合王国
     |公式国名 = {{lang|en|United Kingdom of Great Britain and Northern Ireland}}"

```



これはいまいち。途中で終わってしまっています。




基礎情報の要素の中に `}}` があるので、`}}` でとじられる迄、という検索方法はうまくいきません。かといって



```matlab:Code(Display)
regexp(str,'\{{2}基礎情報.*\}{2}','match')
```



もうまくいきません。


## 前後参照アサーション


ここでは、基礎情報が }} の直前に改行が存在していることを利用します。



```matlab:Code(Display)
（略）
|国際電話番号 = 44↵
|注記 = <references/>↵
}}
```



こんな感じ。




前後参照アサーション（[Link](https://jp.mathworks.com/help/matlab/ref/regexp.html#btn_p45_sep_shared-expression)）と呼ばれる構文を使います。4 種類あります。



   \item{ `expr(?=test): falling `など後ろに `ing` が続く語句と一致させる時に使います。例：\texttt{\w*(?=ing)} など。 }
   -  `expr(?!test): `上の逆で ing が続かない語句との一致。 
   -  `(?<=test)expr: `前に特定の一致を求める場合。例：`(?<=re)\w`* は、`renew` など接頭辞 `re` が付いた文字を探す際に使えますね。 
   -  `(?<!test)expr: `上の逆です。 



ここでは `}}` の前に `\n` (改行) が存在するということにして・・ `(?<=\n) `を追加。



```matlab:Code
baseinfo = regexp(UKWiki.text,'\{{2}基礎情報.*(?<=\n)\}{2}','match')
```


```text:Output
baseinfo = 
    "{{基礎情報 国
     |略名  =イギリス
     |日本語国名 = グレートブリテン及び北アイルランド連合王国
     |公式国名 = {{lang|en|United Kingdom of Great Britain and Northern Ireland}}<ref>英語以外での正式国名:<br />
     *{{lang|gd|An Rìoghachd Aonaichte na Breatainn Mhòr agus Eirinn mu Thuath}}（[[スコットランド・ゲール語]]）
     *{{lang|cy|Teyrnas Gyfunol Prydain Fawr a Gogledd Iwerddon}}（[[ウェールズ語]]）
     *{{lang|ga|Ríocht Aontaithe na Breataine Móire agus Tuaisceart na hÉireann}}（[[アイルランド語]]）
     *{{lang|kw|An Rywvaneth Unys a Vreten Veur hag Iwerdhon Glédh}}（[[コーンウォール語]]）
     *{{lang|sco|Unitit Kinrick o Great Breetain an Northren Ireland}}（[[スコットランド語]]）
     **{{lang|sco|Claught Kängrick o Docht Brätain an Norlin Airlann}}、{{lang|sco|Unitet Kängdom o Great Brittain an Norlin Airlann}}（アルスター・スコットランド語）</ref>
     |国旗画像 = Flag of the United Kingdom.svg
     |国章画像 = [[ファイル:Royal Coat of Arms of the United Kingdom.svg|85px|イギリスの国章]]
     |国章リンク =（[[イギリスの国章|国章]]）
     |標語 = {{lang|fr|[[Dieu et mon droit]]}}<br />（[[フランス語]]:[[Dieu et mon droit|神と我が権利]]）
     |国歌 = [[女王陛下万歳|{{lang|en|God Save the Queen}}]]{{en icon}}<br />''神よ女王を護り賜え''<br />{{center|[[ファイル:United States Navy Band - God Save the Queen.ogg]]}}
     |地図画像 = Europe-UK.svg
     |位置画像 = United Kingdom (+overseas territories) in the World (+Antarctica claims).svg
     |公用語 = [[英語]]
     |首都 = [[ロンドン]]（事実上）
     |最大都市 = ロンドン
     |元首等肩書 = [[イギリスの君主|女王]]
     |元首等氏名 = [[エリザベス2世]]
     |首相等肩書 = [[イギリスの首相|首相]]
     |首相等氏名 = [[ボリス・ジョンソン]]
     |他元首等肩書1 = [[貴族院 (イギリス)|貴族院議長]]
     |他元首等氏名1 = [[:en:Norman Fowler, Baron Fowler|ノーマン・ファウラー]]
     |他元首等肩書2 = [[庶民院 (イギリス)|庶民院議長]]
     |他元首等氏名2 = {{仮リンク|リンゼイ・ホイル|en|Lindsay Hoyle}}
     |他元首等肩書3 = [[連合王国最高裁判所|最高裁判所長官]]
     |他元首等氏名3 = [[:en:Brenda Hale, Baroness Hale of Richmond|ブレンダ・ヘイル]]
     |面積順位 = 76
     |面積大きさ = 1 E11
     |面積値 = 244,820
     |水面積率 = 1.3%
     |人口統計年 = 2018
     |人口順位 = 22
     |人口大きさ = 1 E7
     |人口値 = 6643万5600<ref>{{Cite web|url=https://www.ons.gov.uk/peoplepopulationandcommunity/populationandmigration/populationestimates|title=Population estimates - Office for National Statistics|accessdate=2019-06-26|date=2019-06-26}}</ref>
     |人口密度値 = 271
     |GDP統計年元 = 2012
     |GDP値元 = 1兆5478億<ref name="imf-statistics-gdp">[http://www.imf.org/external/pubs/ft/weo/2012/02/weodata/weorept.aspx?pr.x=70&pr.y=13&sy=2010&ey=2012&scsm=1&ssd=1&sort=country&ds=.&br=1&c=112&s=NGDP%2CNGDPD%2CPPPGDP%2CPPPPC&grp=0&a=IMF>Data and Statistics>World Economic Outlook Databases>By Countrise>United Kingdom]</ref>
     |GDP統計年MER = 2012
     |GDP順位MER = 6
     |GDP値MER = 2兆4337億<ref name="imf-statistics-gdp" />
     |GDP統計年 = 2012
     |GDP順位 = 6
     |GDP値 = 2兆3162億<ref name="imf-statistics-gdp" />
     |GDP/人 = 36,727<ref name="imf-statistics-gdp" />
     |建国形態 = 建国
     |確立形態1 = [[イングランド王国]]／[[スコットランド王国]]<br />（両国とも[[合同法 (1707年)|1707年合同法]]まで）
     |確立年月日1 = 927年／843年
     |確立形態2 = [[グレートブリテン王国]]成立<br />（1707年合同法）
     |確立年月日2 = 1707年{{0}}5月{{0}}1日
     |確立形態3 = [[グレートブリテン及びアイルランド連合王国]]成立<br />（[[合同法 (1800年)|1800年合同法]]）
     |確立年月日3 = 1801年{{0}}1月{{0}}1日
     |確立形態4 = 現在の国号「'''グレートブリテン及び北アイルランド連合王国'''」に変更
     |確立年月日4 = 1927年{{0}}4月12日
     |通貨 = [[スターリング・ポンド|UKポンド]] (£)
     |通貨コード = GBP
     |時間帯 = ±0
     |夏時間 = +1
     |ISO 3166-1 = GB / GBR
     |ccTLD = [[.uk]] / [[.gb]]<ref>使用は.ukに比べ圧倒的少数。</ref>
     |国際電話番号 = 44
     |注記 = <references/>
     }}"

```



うまくいきました。


## フィールド名と値の抽出


ここから `|` のあとの組み合わせ（フィールド名と値）を取得します。= の前後で分割すればいい・・と思いきや




`|GDP/人 = 36,727<ref name="imf-statistics-gdp"/>`




こんな項目があります。




よく見ると (フィールド名）空白 = という形になっているようなので・・



```matlab:Code(Display)
(?<=\n)\|  % 改行文字に続く | 文字 
[^\|\=]*   %　! と = 以外の文字列 0 個以上（フィールド名）
 =         % 空白1つ明けて = 文字
[^\n]*     % 改行以外の何らかの文字列 0 個以上
\n         % 改行
```



でやってみます。



```matlab:Code
regexp(baseinfo,'(?<=\n)\|[^\|\=]* =[^\n]*\n','match')'
```


```text:Output
ans = 58x1 string    
"|略名  =イギリス↵"                  
"|日本語国名 = グレートブリテン及び北アイルランド連…  
"|公式国名 = {{lang|en|United Ki…  
"|国旗画像 = Flag of the United …  
"|国章画像 = [[ファイル:Royal Coat o…  
"|国章リンク =（[[イギリスの国章|国章]]）↵"    
"|標語 = {{lang|fr|[[Dieu et m…  
"|国歌 = [[女王陛下万歳|{{lang|en|Go…  
"|地図画像 = Europe-UK.svg↵"       
"|位置画像 = United Kingdom (+ov…  

```



フィールド名と値をトークン化すると・・



```matlab:Code
baseTokens = regexp(baseinfo,'(?<=\n)\|([^\|\=]*) =([^\n]*)\n','tokens')';
baseTokens{1:2}
```


```text:Output
ans = 1x2 string    
"略名 "        "イギリス"       

ans = 1x2 string    
"日本語国名"      " グレートブリテン及び北アイルランド連合王国"    

```



うまくいっていそう。table 型変数にまとめてみます。



```matlab:Code
fields = cellfun(@(x) x(1), baseTokens);
elements = cellfun(@(x) x(2), baseTokens);
UKinfo = table(fields,elements)
```

| |fields|elements|
|:--:|:--:|:--:|
|1|"略名 "|"イギリス"|
|2|"日本語国名"|" グレートブリテン及び北アイルランド連合王国"|
|3|"公式国名"|" {{lang|en|United Kingdom of Great Britain and Northern Ireland\|
|4|"国旗画像"|" Flag of the United Kingdom.svg"|
|5|"国章画像"|" [[ファイル:Royal Coat of Arms of the United Kingdom.svg|85px|イギリスの国章]]"|
|6|"国章リンク"|"（[[イギリスの国章|国章]]）"|
|7|"標語"|" {{lang|fr|[[Dieu et mon droit]]\|
|8|"国歌"|" [[女王陛下万歳|{{lang|en|God Save the Queen\|
|9|"地図画像"|" Europe-UK.svg"|
|10|"位置画像"|" United Kingdom (+overseas territories) in the World (+Antarctica claims).svg"|
|11|"公用語"|" [[英語]]"|
|12|"首都"|" [[ロンドン]]（事実上）"|
|13|"最大都市"|" ロンドン"|
|14|"元首等肩書"|" [[イギリスの君主|女王]]"|

## 辞書オブジェクトとは？ 


要は「キーと値のペアを登録できるデータ構造」みたいなので、機能を果たしそうな [containers.Map 値を一意のキーにマップするオブジェクト](https://jp.mathworks.com/help/matlab/ref/containers.map.html) を使ってみます。



```matlab:Code
M = containers.Map(fields,elements);
M('首相等氏名')
```


```text:Output
ans = ' [[ボリス・ジョンソン]]'
```

# 26. 強調マークアップの除去
> 25の処理時に，テンプレートの値からMediaWikiの強調マークアップ（弱い強調，強調，強い強調のすべて）を除去してテキストに変換せよ（参考: [マークアップ早見表](http://ja.wikipedia.org/wiki/Help:%E6%97%A9%E8%A6%8B%E8%A1%A8)）．


  


例えばこれ：'''グレートブリテン及び北アイルランド連合王国'''




このはじめと終わりにあるのが強調マークアップです。マークアップ早見表によると



   -  他との区別（斜体）：''他との区別'' 
   -  強調（太字）：'''強調''' 
   -  斜体と強調：'''''斜体と強調''''' 



ということで2個、3個、5個のケースがある模様。



```matlab:Code
samplestr = '''''グレートブリテン及び北アイルランド連合王国''''';
tokens = regexp(samplestr,'(\''{2,5})(.*?)(\1)','tokens');
tokens{:}
```


```text:Output
ans = 1x3 cell    
''''         'グレートブリテン及び北アイルランド連合王国'    ''''         

```



2つ目のトークンとして取られていますね。[regexprep](https://jp.mathworks.com/help/matlab/ref/regexprep.html) 関数を使うと便利そう。




これは正規表現を使った文字置換ができます。置換文字を直接指定もできますし、以下の例だと $2 と2番目のトークンで置き換える、という処理をしています。今回のケースだと 2 つめのトークンで置換してみると、



```matlab:Code
regexprep(samplestr,'(\''{2,5})(.*?)(\1)','$2')
```


```text:Output
ans = 'グレートブリテン及び北アイルランド連合王国'
```



`UKinfo` への処理は 27 と一緒に 28 で実施します。


# 27. 内部リンクの除去
> 26の処理に加えて，テンプレートの値からMediaWikiの内部リンクマークアップを除去し，テキストに変換せよ（参考: [マークアップ早見表](http://ja.wikipedia.org/wiki/Help:%E6%97%A9%E8%A6%8B%E8%A1%A8)）．


  


マークアップ早見表によると内部リンクには



   -  [[記事名]] 
   -  [[記事名|表示文字]] 
   -  [[記事名\#節名|表示文字]] 



の3種類がある模様。例えば: [[ボリス・ジョンソン]] は簡単です。かっこの中を単純に取り出せばOK。



```matlab:Code
samplestr = '[[ボリス・ジョンソン]]';
tokens = regexp(samplestr,'\[{2}(.*)\]{2}','tokens');
tokens{1}
```


```text:Output
ans = 
    {'ボリス・ジョンソン'}

```



記事名と表示文字がある場合は表示文字だけを残します。`|` のあとをトークン化。



```matlab:Code(Display)
\[{2}    % [ 2個
[^\|]+   % | 以外の文字が 1 個以上で、
\|       % | が 1 個あった後に
([^\|]+) % | 以外の文字が 1 個以上 （トークン化）
\]{2}    % ] 2個
```


```matlab:Code
samplestr = '[[イギリスの国章|国章]]';
tokens = regexp(samplestr,'\[{2}[^\|]+\|([^\|]+)\]{2}','tokens');
tokens{1}
```


```text:Output
ans = 
    {'国章'}

```

## **条件演算子**


ボリス・ジョンソンなど「記事名」だけの場合と、表示文字がある場合で同じ正規表現にしたい・・。ただいい手が思いつかないので、単純に `expr1|expr2 `と組み合わせてみます。この場合 `expr1` と一致する場合、`expr2` は無視されます。



```matlab:Code
expr1 = '\[{2}[^\|]+\|([^\|]+)\]{2}';
expr2 = '\[{2}(.*)\]{2}';

samplestr = '[[ボリス・ジョンソン]]';
tokens = regexp(samplestr,[expr1,'|',expr2],'tokens');
tokens{1}
```


```text:Output
ans = 
    {'ボリス・ジョンソン'}

```


```matlab:Code
samplestr = '[[イギリスの国章|国章]]';
tokens = regexp(samplestr,[expr1,'|',expr2],'tokens');
tokens{1}
```


```text:Output
ans = 
    {'国章'}

```



いい感じ。


## regexprep 関数で置換


ここも [regexprep](https://jp.mathworks.com/help/matlab/ref/regexprep.html) 関数を使って文字置換してみます。



```matlab:Code
samplestr = '[[イギリスの国章|国章]]';
regexprep(samplestr,[expr1,'|',expr2],'$1')
```


```text:Output
ans = '国章'
```


```matlab:Code
samplestr = '[[ボリス・ジョンソン]]';
regexprep(samplestr,[expr1,'|',expr2],'$1')
```


```text:Output
ans = 'ボリス・ジョンソン'
```



`UKinfo` への処理は 26 と一緒に 28 で実施します。


# 28. MediaWikiマークアップの除去
> 27の処理に加えて，テンプレートの値からMediaWikiマークアップを可能な限り除去し，国の基本情報を整形せよ．


  


他に何があるのか・・。`UKinfo` を目で確認しながらそれっぽいものを処理していきます。24, 26, 27 の処理と合わせて実施します。



```matlab:Code
baseinfo = regexp(UKWiki.text,'\{{2}基礎情報.*(?<=\n)\}{2}','match');

% <ref name=　*</ref>, <ref name=　* />, <br /> の除去
%
% 正規表現
% <        : <
% \/?      : / が 0 回か 1 回
% [br|ref] : br もしくは ref
% [^>]*?   : > 以外の文字を 0 回以上（非貪欲）
% >        : >
baseinfo1 = regexprep(baseinfo,'<\/?[br|ref][^>]*?>','');

% 参照：24. ファイル参照の抽出（ファイル名だけ残します。）
baseinfo1 = regexprep(baseinfo1,'\[{2}ファイル:([^\]\|]*)(?:\]{2}|\|[^\]\|]*)+','$1');

% 参照：26. 強調マークアップの除去
baseinfo1 = regexprep(baseinfo1,'(\''{2,5})(.*?)(\1)','$2');

% 参照：27. 内部リンクの除去
expr1 = '\[{2}[^\|\]]+\|([^\|\]]+)\]{2}';
expr2 = '\[{2}([^\[\]]+)\]{2}';
baseinfo1 = regexprep(baseinfo1,[expr1,'|',expr2],'$1');

% 追加：lang テンプレートの除去 {{lang|言語タグ|文字列}} 文字列だけを残します。
%
% 正規表現
% \{{2}        : { 2 回
% lang         : lang 文字列
% \|           : | 1 回
% \w+          : 任意の文字 1 回以上
% \|           : | 1 回
% ([^\}\|]*)   : } と | 以外の文字 0 回以上（トークン化）
% \}{2}        : } 2 回
baseinfo1 = regexprep(baseinfo1,'\{{2}lang\|\w+\|([^\}\|]*)\}{2}','$1');
```



この時点でさらに以下の MediaWiki マークアップのようなものがみられますので、個別に処理します。



```matlab:Code(Display)
|国歌 = 女王陛下万歳|God Save the Queen{{en icon}} 神よ女王を護り賜え {{center|United States Navy Band - God Save the Queen.ogg}}
|他元首等氏名2 = {{仮リンク|リンゼイ・ホイル|en|Lindsay Hoyle}}
|人口値 = 6643万5600 {{Cite web|url=https://www.o
|GDP値元 = 1兆5478億 [http://www.imf.org/ex
|確立年月日2 = 1707年{{0}}5月{{0}}1日
```


```matlab:Code
%　|国歌 = 女王陛下万歳|God Save the Queen{{en icon}} 
% 神よ女王を護り賜え {{center|United States Navy Band - God Save the Queen.ogg}}
baseinfo1 = regexprep(baseinfo1,'\{{2}en icon\}{2}','');
baseinfo1 = regexprep(baseinfo1,'\{{2}center\|([^\}]+)\}{2}','$1');

% |他元首等氏名2 = {{仮リンク|リンゼイ・ホイル|en|Lindsay Hoyle}}
% 参照：24. ファイル参照の抽出
baseinfo1 = regexprep(baseinfo1,'\{{2}仮リンク\|([^\}\|]*)(?:\}{2}|\|[^\}\|]*)+','$1');

% |人口値 = 6643万5600 {{Cite web|url=https://www.o
% |GDP値元 = 1兆5478億 [http://www.imf.org/ex
baseinfo1 = regexprep(baseinfo1,'\[http:.*\]','');
baseinfo1 = regexprep(baseinfo1,'\{{2}Cite web\|url\=https:[^\}]+\}{2}','');

% |確立年月日2 = 1707年{{0}}5月{{0}}1日
baseinfo1 = regexprep(baseinfo1,'\{{2}(\d)\}{2}','$1');
```



以上！長かった。。




結果をテーブルにまとめておきます。



```matlab:Code
baseTokens = regexp(baseinfo1,'(?<=\n)\|([^\|\=]*) =\s?([^\|]*)\n','tokens')';
fields = cellfun(@(x) x(1), baseTokens);
elements = cellfun(@(x) x(2), baseTokens);
table(fields,elements)
```

| |fields|elements|
|:--:|:--:|:--:|
|1|"略名 "|"イギリス"|
|2|"日本語国名"|"グレートブリテン及び北アイルランド連合王国"|
|3|"公式国名"|



Map オブジェクトも作っておきます。



```matlab:Code
M = containers.Map(fields,elements);
M('国旗画像')
```


```text:Output
ans = 'Flag of the United Kingdom.svg'
```

# 29. 国旗画像のURLを取得する
> テンプレートの内容を利用し，国旗画像のURLを取得せよ．（ヒント: [MediaWiki API](http://www.mediawiki.org/wiki/API:Main_page/ja)の[imageinfo](https://www.mediawiki.org/wiki/API:Imageinfo)を呼び出して，ファイル参照をURLに変換すればよい）


  


上の結果より、国旗画像のファイル名は `Flag of the United Kingdom.svg` ということでした。




MediaWiki APIの [imageinfo](https://www.mediawiki.org/wiki/API:Imageinfo) にある GET request の箇所を確認すると、以下のフォーマットでファイルの情報が取れるとあります。


> api.php?action=query\&format=json\&prop=imageinfo\&titles=File:Billy_Tipton.jpg 




URL を取るには、追加で iiprop に url を入れておけば良いようです。MATLAB では webread 関数を使えばいいですね。



```matlab:Code
baseurl = 'https://commons.wikimedia.org/w/api.php';

data = webread(baseurl,'action','query',...
    'format','json',...
    'prop','imageinfo',...
    'iiprop','url',...
    'titles','File:Flag_of_the_United_Kingdom.svg');

data.query.pages.x347935.imageinfo
```


```text:Output
ans = 
                    url: 'https://upload.wikimedia.org/wikipedia/commons/a/ae/Flag_of_the_United_Kingdom.svg'
         descriptionurl: 'https://commons.wikimedia.org/wiki/File:Flag_of_the_United_Kingdom.svg'
    descriptionshorturl: 'https://commons.wikimedia.org/w/index.php?curid=347935'

```



URL とれました。


