---
title: 学習なしPhi-2でARCを解いてみた。
tags:
  - Python
  - Phi_2
  - transformers
  - AbstractReasoningChallenge
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

このプロンプトを使用すると、おおよそ回答の数字の羅列 or 数字の羅列＋その羅列を作成するためのC言語系のコードと言う形が多かった。  
この返答に対して２行以上改行されているところまでをPhi-2の回答とし、その回答と
本当の答えが同一かを判定することで今回の正解数を計測した。

## 正解数と正解したタスク

### 正解数

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

1. 計数 → 1つの色を回答(画像1~3)
2. パターンの拡張・縮小と回転(画像4~8)

上記の２つに分けられると考えられる。  
正解したタスクは２パターンしかないが、実際のARCには他にも色々な種類のタスクがある。
またパターン1は１つの色を回答するだけなので、パターン２はタスク内のパターンを移動するだけのARCの中でも簡単なタスクだと考えられる。  
すなわちARCの思考力はまだ至らない部分がかなりあると考えられる。

### trainingしか正解しなかった原因

原因としては

1. trainingのみがPhi-2の学習データ元に入っていた。
2. evaluationがtrainingより難しい。

の２つが考えられる。  
ただし、2のほうが有り得そうと思っている。
わざわざtrainingだけにフィルタリングをして学習データ元を作成するとは考えづらいし
私が見た限りでは全体的にevaluationはtrainingより難しい。
１問も解けなくてもおかしくないと考えている。

## その他

### Phi-2の回答のクセ

他にもいくつかのプロンプトの形式を試したが、文章のようなものを入れ込んでしまうと文章やC言語系のコードだけで回答されることが多かった。
コードで回答される事が多い理由はおそらくPhi-2のトレーニングにも
コードが多く、特にこのような数字の羅列とプログラム以外にはなかなか
ないので、そのような回答をしたのかと。
ただ、そのような形式で受け取ってしまうと処理が難しいので
数字と記号とインデックスだけの[プロンプト](#プロンプト)を採用した。

AIの面倒を見るのも中々、大変で。

