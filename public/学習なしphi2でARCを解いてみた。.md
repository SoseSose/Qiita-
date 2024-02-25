---
title: 学習なしPhi2でARCを解いてみた。
tags:
  - ''
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

## 目的  

何も学習させていないPhi-2でARCを学習させるとどれくらいの正解率になるのか？  
また、どのような種類の問題が解けるのかを大まかに示す。

## Phi-2以外のモデルの正解率

下記のサイトで計測されている。やはりGPT-4は性能が高く
21%の正解率となっている。

https://github.com/alxndrTL/ARC_LLMs

## 計測したコード

自作リポジトリのarc_evaluate_phi2.pyで推論を実行し、arc_phi2_analyze.pyで正解率と正解したタスクの視覚化を実装した。

https://github.com/SoseSose/phi-2-train

## プロンプト

次のようなプロンプトを採用した。  
以下はタスク017c7c7bでの例。

```txt
-example0-
010
110
010
011
010
110
->
020
220
020
022
020
220
020
022
020

-example1-
010
101
010
101
010
101
->
020
202
020
202
020
202
020
202
020

-example2-
010
110
010
010
110
010
->
020
220
020
020
220
020
020
220
020

-test-

111
010
010
111
010
010
->
```

回答は次のような形で返ってくる。

```txt
020
202
020
202
020
202
020
202
020

A:

I think you are looking for something like this:
#include <stdio.h>

int main(void) {
    int a[10][10];
    int i, j, k;
    int n = 10;

    for (i = 0; i < n; i++) {
        for (j = 0; j < n; j++) {
            a[i][j] = 0;
        }
    }
...
    return 0;
}
```

おおよそ回答の数字の羅列 or 数字の羅列＋その羅列を作成するためのC言語系のコードと言う形が多かった。
この返答に対して２行以上改行されているところまでをPhi-2の回答とし、その回答と
本当の答えが同一かを判定することで今回の正解数を計測した。

## 正解数と正解したタスク


|  | 正解数| タスク数 |
| ---- | ---- | ---- |
| training | 8 | 400 |
| evaluation | 0 | 400 |

### 正解した画像

#### 画像1　(445eab21)

![445eab21.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/595608/f9ab3fda-b75c-5705-cd62-a77fb60fa1ab.png)

#### 画像2　(44f52bb0)

![44f52bb0.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/595608/d23b1729-8300-5797-5b4e-322c94bb337b.png)

#### 画像3　(239be575)

![239be575.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/595608/d081616f-ca36-a702-c42e-72525fec1992.png)

#### 画像4　(7b7f7511)

![7b7f7511.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/595608/55154002-68f5-db1a-c2e1-968e12323e2b.png)

#### 画像5　(6fa7a44f)

![6fa7a44f.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/595608/faf08eb2-855f-6ab4-da82-f122780d661f.png)

#### 画像6　(53b68214)

![53b68214.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/595608/88411e9c-c681-6aeb-5515-64e1ffa616ad.png)

#### 画像7　(68b16354)

![68b16354.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/595608/b80afb89-f8de-ea73-1031-f301f002a904.png)

#### 画像8　(8be77c9e)

![8be77c9e.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/595608/c1359819-d61c-7f0d-f040-d5f9591d843d.png)

## 考察

### 正解したタスクのパターン

1. 計数(画像1~3)
2. パターンの拡張・縮小と回転(画像4~8)

に分けられると考えられる。

### trainingしか正解しなかった原因

原因としては

1.
2.

の２つが考えられる。

## Phi-2の回答のクセ

他にもいくつかのプロンプトの形式を試したが、文章のようなものを入れ込んでしまうと文章やコードで回答されることが多かった。

またコードで回答されることも多かった、おそらくPhi-2のトレーニングにも
コードが多く、特にこのような数字の羅列とプログラム以外にはなかなか
ないので、そのような回答をしたのかと。
ただ、このような形式で受け取ってしまうと処理が難しい(C++の処理系を組み込まないと)ので
数字と記号とインデックスだけを使用して、次のようなプロンプトにした。

結果、まずARCの答えを返答し、その後にその答えを生成するプログラムコードを記載するという
形式を取ってくれるように。

AIの面倒を見るのも中々、大変で。
