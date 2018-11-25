# qtq.Merge
qtq.Mergeは、「クイックソートより速い（Quicker than QuickSort）」を目指した、マージソートの改良アルゴリズムです。

以下の特徴があります。  
* 比較ソート
* 安定ソート
* 外部ソート：N
* 平均時間：O(N log N)
* 最悪時間：O(N log N)
* 昇順ソート済み：O(N)
* 降順ソート済み：O(2N)
* 再帰無し

<br>

# ビルド＆テスト
是非、あなたの環境でも試してみてください。

## **Msvc**
cl Main.cpp -Ox -EHsc -Fe:TestMsvc.exe  
TestMsvc.exe  

## **clang++**
clang++ Main.cpp -Ofast -o TestClang.exe  
TestClang.exe  

## **g++**
g++ Main.cpp -Ofast -o TestGcc.exe  
TestGcc.exe  

<br>

# 評価サンプル
動作環境は以下の通り。  
  * Windows 10 Pro 64bit  
  * Core i7-8700 3.20GHz  

同じシードから生成した、ランダムなfloat値をソートしてみました。  
単位は秒で、数値が低いほど高速です。

## **Msvc**
Microsoft(R) C/C++ Optimizing Compiler Version 19.15.26732.1 for x64  
Microsoft (R) Incremental Linker Version 14.15.26732.1  

|数|std::sort|std::stable_sort|qtq.Merge.TypeV|
|-:|-:|-:|-:|
|10,000|0.00083858|*0.00069774*|**0.00060311**|
|1,000,000|0.07069080|*0.06401268*|**0.06192394**|
|100,000,000|8.96780856|*8.58470773*|**8.15964417**|
他のコンパイラと比較しても、std::sortが遅く、std::stable_sortが速い結果となったMsvc。  
初っ端からかましてくれましたが、この特性のお陰で勝つことができました。  

## **clang**
clang version 7.0.0 (tags/RELEASE_700/final)  
Target: x86_64-w64-windows-gnu  

|数|std::sort|std::stable_sort|qtq.Merge.TypeV|
|-:|-:|-:|-:|
|10,000|*0.00041480*|0.00045297|**0.00040581**|
|1,000,000|**0.06034432**|0.06799446|*0.06345384*|
|100,000,000|**7.61524030**|9.22656715|*8.38774283*|
今一つぱっとしないclang。  
ソースレベルで最適化を行う必要があるなど、コンパイラの最適化ロジックに疑問が残る結果となりました。  

## **gcc**
gcc version 8.2.0 (Rev3, Built by MSYS2 project)  
Target: x86_64-w64-mingw32  

|数|std::sort|std::stable_sort|qtq.Merge.TypeV|
|-:|-:|-:|-:|
|10,000|0.00041768|0.00045425|**0.00038624**|
|1,000,000|**0.06043607**|0.06795885|*0.06112900*|
|100,000,000|**7.82155688**|9.08399603|*7.91397587*|
善戦できたのがgcc。  
別のシードから生成したランダムでは勝てる場合もあり、拮抗した結果となりました。  

<br>

# 基本となるアルゴリズム

* 外部領域を、2Nの連続帯として見立てます。
* 値を外部領域に置く際、以下のルールを適用します。
  * （最大値 ≦ 値）であれば、昇順エリアの上側に置き、最大値を更新します。
  * （値 ＜ 最小値）であれば、降順エリアの下側に置き、最小値を更新します。
  * （最小値 ≦ 値 ＜ 最大値）であれば、新しい値（最大値であり最小値）を昇順エリアに置き、以前に並べた値群をPartとします。

## 具体的な流れ
~~~
                入力列
               |4 5 1 2 7 6 3 8
. . . . . . . . . . . . . . . .
    降順エリア<-|->昇順エリア
~~~
~~~
新しい値（最大値であり最小値）を昇順エリアに置く
               |. 5 1 2 7 6 3 8
. . . . . . . . 4 . . . . . . .
~~~
~~~
次の値は（最大値 ≦ 値）なので、昇順エリアに置く
               |. . 1 2 7 6 3 8
. . . . . . . . 4 5 . . . . . .
~~~
~~~
次の値は（値 ＜ 最小値）なので、降順エリアに置く
               |. . . 2 7 6 3 8
. . . . . . . 1 4 5 . . . . . .
~~~
~~~
次の値は（最小値 ≦ 値 ＜ 最大値）なので、新しい値を昇順エリアに置く（Part：145が完成）
               |. . . . 7 6 3 8
. . . . . . .|1 4 5|2 . . . . .
~~~
~~~
次の値は（最大値 ≦ 値）なので、昇順エリアに置く
               |. . . . . 6 3 8
. . . . . . .|1 4 5|2 7 . . . .
~~~
~~~
次の値は（最小値 ≦ 値 ＜ 最大値）なので、新しい値を昇順エリアに置く（Part：27が完成）
               |. . . . . . 3 8
. . . . . . .|1 4 5|2 7|6 . . .
~~~
~~~
次の値は（値 ＜ 最小値）なので、降順エリアに置く
               |. . . . . . . 8
. . . . . . 3|1 4 5|2 7|6 . . .
~~~
~~~
次の値は（最大値 ≦ 値）なので、昇順エリアに置く（Part：368が完成）
               |. . . . . . . .
. . . . . .|3|1 4 5|2 7|6 8|. .
~~~
~~~
最終的な外部領域
4 5|2 7|6 8|. .  昇順エリア見立て
. . . . . .|3|1  降順エリア見立て
4 5|2 7|6 8|3|1  実際の内容
~~~
~~~
生成したPart群をマージします  
145 27 368  
12457 368  
12345678  
ソート完了  
~~~

<br>

# 改良点
基本アルゴリズムから更に、以下の改良を施してあります。
* Partの長さを確保する為、インサートソートを行っています。
* 再帰をしないように、順次マージを行っています。

<br>

# 余談
如何だったでしょうか？  

実際には、スカラー値のみでソートをする場面は殆どなく、実用においてクイックソート種に勝つことは難しくなります。  

ですが、未来がどうなっているか分かりません。  
今後、実用レベルにおいて、マージソート種がクイックソート種に勝る日は来るでしょうか？  
今回、その可能性を示唆することが出来たのであれば、本望です。  

ソートアルゴリズムには、まだ浪漫が残っています。  
