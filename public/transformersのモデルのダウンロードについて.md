---
title: transformersのモデルのダウンロードについて
tags:
  - Python
  - transformers
private: false
updated_at: '2024-02-22T02:50:02+09:00'
id: 00185f49f245c4d8df16
organization_url_name: null
slide: false
ignorePublish: false
---
## 目的  

transformersのモデルをローカルにダウンロードする方法を示す。

## 動機

transformersで作業を続けているとき、アップデートが
入ると再ダウンロードが必要になる。Phi-2で約5GBの保存容量が必要になり
ダウンロードの度にそれなりの時間がかかり煩わしい。(光回線が引けない家🥺)

## コード

AutoTokenizerやAutoModelForCausalLMのsave_pretrained関数のsave_directoryという引数に
保存先を指定するとモデルを指定場所に保存できる。
保存したあとはfrom_pretrainedのpretrained_model_name_or_path引数に保存した場所を指定すれば
モデルを読み込み使用できる。

```python

from pathlib import Path
from transformers import AutoTokenizer, AutoModelForCausalLM

def get_tokenizer_and_model(pre_trained_model_name, save_dir)
    if not Path(save_dir).exists():
        # download model
        self.tokenizer = AutoTokenizer.from_pretrained(pre_trained_model_name)
        self.model = AutoModelForCausalLM.from_pretrained(pre_trained_model_name)
        # save model
        self.tokenizer.save_pretrained(save_dir)
        self.model.save_pretrained(save_dir)

    else:
        # load model
        tokenizer = AutoTokenizer.from_pretrained(save_dir)
        model = AutoModelForCausalLM.from_pretrained(save_dir)

```  

## まとめ

時々入る再ダウンロードに煩わされることがなく快適に!!
