### **Gemini API 入門：Python による生成 AI アプリケーション開発体験**

#### **1. はじめに**

近年，ChatGPT に代表される生成 AI (Artificial Intelligence) が注目を集めている．しかし，「ChatGPT」という名称が指すものは，ユーザーが対話するためのユーザーインターフェース (User Interface, UI) であり，その背後で動作する大規模言語モデル (Large Language Model, LLM) そのものではない．例えば OpenAI 社が提供するサービスでは，ChatGPT という UI を通じて，GPT-4o や GPT-4.1 mini といった LLM を利用する．

この関係は，Google 社が提供する生成 AI でも同様である．Web ブラウザ上で利用できる Google AI Studio は UI であり，その内部では Gemini と呼ばれる LLM が動作している．

本演習では，生成 AI の基本的な構造を理解するため，Google の Gemini を題材として，プログラムから LLM を直接利用する方法を体験する．具体的には，Python というプログラミング言語と，Gemini を操作するためのソフトウェア開発キット (Software Development Kit, SDK) を用いて，簡易的なアプリケーション開発を体験する．

#### **2. 演習の構成**

本演習は以下の手順で実施する．

1.  **Google AI Studio の利用**: まずは Web UI を通じて Gemini の基本的な性能を体験する．
2.  **API キーの取得**: プログラムから Gemini を利用するために必要な認証キーを取得する．
3.  **開発環境の準備**: Python プログラムを実行するための環境を構築する．
4.  **Python による API の利用**: 実際にコードを記述し，Gemini との対話を自動化する．
5.  **応用**: モデルの性能や挙動を変化させるパラメータを調整する．

#### **3. 演習 1：Google AI Studio で Gemini を体験する**

はじめに，Web UI である Google AI Studio を利用して，Gemini がどのようなものかを直感的に理解する．

1.  Web ブラウザで Google AI Studio ([https://aistudio.google.com/](https://aistudio.google.com/)) にアクセスし，自身の Google アカウントでログインする．
2.  チャット形式の入力画面が表示されれば準備完了である．この画面は ChatGPT の UI と類似しており，プロンプト（指示文）を入力することで，AI との対話が可能である．
3.  いくつかの質問を投げかけ，その応答を確認する．

##### **3.1 AI Studio を学習の補助として活用する**

この演習を進める中で，もし分からない専門用語（例：API, SDK）やプログラミングの概念が出てきた場合は，この AI Studio に直接質問することが推奨される．例えば，「APIとは何ですか？小学生にも分かるように説明してください」のように入力することで，対話形式で知識を補うことが可能である．AI Studio は，本演習における強力な学習アシスタントとなる．

例えば，この演習資料で使われている「Markdown (マークダウン)」という文書形式が分からない場合でも，AI Studio に「Markdown がわからないので基本を簡潔に教えて」と質問することで，速やかに学習できる．

また，AI Studio の画面右上では，対話に使用する LLM のモデルを選択できる．高性能な「Gemini 2.5 Pro」は質の高い応答を生成するが，無料利用枠の制限に達しやすい場合がある．もし利用できなくなった場合は，より軽量な「Gemini 1.5 Flash」に切り替えるか，一日待つことで利用枠が回復するため，焦らずに対応することが望ましい．

#### **4. 演習 2：Python から Gemini API を利用する**

次に，Web UI を介さず，Python プログラムから直接 Gemini を呼び出す方法を学ぶ．これにより，定型的なタスクの自動化や，他のアプリケーションとの連携が可能となる．このプログラムと LLM の仲立ちをする仕組みを，アプリケーションプログラミングインターフェース (Application Programming Interface, API) と呼ぶ．

##### **4.1 API キーの取得**

API を利用するには，身分証明書に相当する API キーが必要となる．

1.  Google AI Studio の画面左側にあるメニューから「Get API key」を選択する．
2.  「Create API key」ボタンをクリックし，API キーを生成する．
3.  `AIza...` から始まる文字列が API キーである．このキーは他者に知られないよう，テキストエディタなどに一時的にコピーし，安全な場所に保管すること．

##### **4.2 開発環境の準備**

Python プログラムを実行する環境を準備する．本演習では，Web ブラウザ上で環境構築からコード実行までを完結できる Google Colaboratory (以下，Colab) の利用を推奨する．

**A) Google Colaboratory を利用する場合 (推奨)**

1.  Google Colab ([https://colab.research.google.com/](https://colab.research.google.com/)) にアクセスし，「ノートブックを新規作成」をクリックする．
2.  はじめに，Gemini API を利用するためのライブラリをインストールする．以下のコマンドをコードセルに入力し，実行する．

    ```bash
    !pip install -q google-genai
    ```

3.  次に，取得した API キーを設定する．Colab の画面左側にある鍵アイコン（シークレット）をクリックし，「新しいシークレットを追加」を選択する．名前に `GEMINI_API_KEY`，値に先ほどコピーした API キーを入力し，「このノートブックでアクセスを有効にする」のトグルをオンにする．これにより，キーをコード中に直接記述することなく安全に利用できる．

**B) ローカル環境 (Anaconda) を利用する場合**

自身の PC に Anaconda がインストールされている場合は，以下の手順で仮想環境を構築する．

1.  ターミナル（Windows の場合は Anaconda Prompt）を開き，以下のコマンドで仮想環境を作成し，有効化する．

    ```bash
    conda create -n gemini-env python=3.11 -y
    conda activate gemini-env
    ```

2.  ライブラリをインストールする．

    ```bash
    pip install google-genai
    ```

3.  API キーを環境変数として設定する．この設定方法は OS によって異なる．（YOUR_API_KEY の部分を自身のキーに置き換えること）
    *   **macOS/Linux (Bash/Zsh)**: `export GEMINI_API_KEY='YOUR_API_KEY'`
    *   **Windows (コマンドプロンプト)**: `setx GEMINI_API_KEY "YOUR_API_KEY"`
    *   恒久的な設定には，`~/.bashrc` やシステムのプロパティからの設定が必要となる．

##### **4.3 動作確認コードの実行**

環境が整ったら，最小限の Python コードで API が正しく呼び出せるかを確認する．以下のコードを Colab の新しいセル，またはローカルのテキストファイル (`gemini_test.py` など) として保存し，実行する．

```python
import os
import google.generativeai as genai

# API キーを設定 (Colab の場合は Secrets から自動で読み込まれる)
# ローカル環境で環境変数を設定した場合も同様
api_key = os.getenv("GEMINI_API_KEY")
genai.configure(api_key=api_key)

# 使用するモデルを選択
# gemini-1.5-flash は無料枠で安定して使える高速モデル
model = genai.GenerativeModel("gemini-1.5-flash")
# gemini-2.5-flash は最新の高速モデル（API利用料に注意を要する）
# model = genai.GenerativeModel("gemini-2.5-flash")
# gemini-2.5-pro は最高性能モデル（API利用料に注意を要する）
# model = genai.GenerativeModel("gemini-2.5-pro")

# LLM にプロンプトを送信し，応答を生成
response = model.generate_content("日本の首都について，簡潔に説明してください．")

# 応答からテキスト部分のみを抽出して表示
print(response.text)
```

実行後，コンソールに「日本の首都は東京です．...」といった応答が表示されれば成功である．
例えば，「日本の首都について，簡潔に説明してください．」の部分を「日本の首都について，カエルの精霊娘っぽく説明してください．」に変更して実行した場合の挙動を確認し，それ以外にも質問内容を変更してみること．また，上記プログラムの実行でエラーが出た場合は，エラーメッセージを AI Studio に入力することで解決策を各自で確認すること．

#### **5. 応用演習：モデルとパラメータの調整**

API を利用する利点の一つは，モデルの種類やその挙動を細かく制御できる点にある．

##### **5.1 モデルの切り替え**

Gemini API では，用途やコストに応じて複数のモデルを使い分けることが可能である．

*   **`Gemini 2.5 Pro`**: 性能を最も重視する場合の最上位モデルである．複雑な推論や質の高い文章生成など，高度な処理が求められるタスクに適している．
*   **`Gemini 2.5 Flash`**: 速度と効率に特化した最新の軽量モデルである．応答が非常に速いため，チャットボットのようなリアルタイム性が重要なアプリケーションに向いている．
*   **`Gemini 1.5 Flash`**: API の無料利用枠で安定して利用できる，信頼性の高いモデルである．最新の 2.5 系列が登場したが，コストを抑えたい場合や，多くのチュートリアルで採用されている実績のあるモデルとして，依然として重要な選択肢である．

モデルを切り替えるには，コード中のモデル ID を書き換えるだけで実現可能である．

```python
# モデルを切り替えるには，コード中のモデル ID を書き換える．

# 高性能モデル (Pro) を利用する場合
# model = genai.GenerativeModel("gemini-2.5-pro")

# 最新の高速モデル (Flash) を利用する場合
# model = genai.GenerativeModel("gemini-2.5-flash")

# 無料枠で利用しやすい安定版モデルを利用する場合
model = genai.GenerativeModel("gemini-1.5-flash")
```

同じプロンプト（例：50 語程度の英文の翻訳）をこれらのモデルで実行し，応答の品質，生成速度，そして API の応答時間にどのような違いが現れるか比較されたい．なお，モデル選択画面には「Preview」と付くモデルも存在する．これらは新機能などを先行して試せる実験的なバージョンであるが，本演習では安定版である上記のモデルを利用することを推奨する．どのようなモデルが使用可能か AI Studio を用いて各自確認すること．

##### **5.2 パラメータの調整**

LLM の応答は，その生成プロセスを制御する複数のパラメータによって調整できる．Gemini API では，これらの設定を `GenerationConfig` クラスに集約して指定する．これにより，応答の多様性や一貫性を細かく制御することが可能となる．

```python
import os
import google.generativeai as genai

# API キーを設定
api_key = os.getenv("GEMINI_API_KEY")
genai.configure(api_key=api_key)

model = genai.GenerativeModel("gemini-1.5-flash")

# 応答の多様性や一貫性を制御するパラメータ群
generation_config = genai.types.GenerationConfig(
    # 応答のランダム性を指定．値が大きいほど多様になる．
    temperature=0.9,
    # 候補トークンを累積確率で絞り込む (Nucleus Sampling)
    top_p=0.95,
    # 候補トークンを上位k個に絞り込む
    top_k=40,
    # 生成するテキストの最大長（トークン数）
    max_output_tokens=200
)

response = model.generate_content(
    "面白い物語の冒頭を考えてください．",
    generation_config=generation_config
)
print(response.text)
```

主要なパラメータの役割は以下の通りである．これは Gemini 固有ではなく一般的な LLM においても用いられているパラメータである．

*   **`temperature`**: 応答のランダム性を制御する．0 に近いほどモデルが予測した確率の高い応答に近づき（決定的になる傾向がある），1 に近いほど多様で創造的な応答が生成されやすくなる．サンプリングを伴う `top_k` や `top_p` を有効にするには，0 より大きい値を設定する必要がある．

*   **`top_k`**: 次に出力するトークン（単語や文字の一部）の候補を，確率が高い上位 `k` 個に限定する．例えば `top_k=40` とした場合，40個の候補の中から次のトークンが選ばれる．これにより，極端に確率の低い不自然なトークンの出現を抑制できる．

*   **`top_p` (Nucleus Sampling)**: 次のトークン候補を，確率の高いものから順に足し合わせ，その累積確率が `p` を超えるまでの集合に限定する．例えば `top_p=0.95` とした場合，確率の合計が 95% に達するまでの最小限のトークン群が候補となる．文脈に応じて候補数が動的に変化するため，より自然で高品質な文章生成が期待できる．

*   **`max_output_tokens`**: 生成されるテキストの最大長をトークン数で制限する．

これらのパラメータを様々に組合せ，出力がどのように変化するかを観察されたい．例えば，創造的な文章を生成したい場合は `temperature` と `top_p` を高めに設定し，より事実に基づいた応答を求める場合は `temperature` を低く設定するなどの調整が有効である．

#### **6. まとめ**

本演習を通じて，生成 AI における UI と LLM の違い，そして API を用いてプログラムから LLM を利用する基本的な手法を学んだ．具体的には，Google AI Studio での対話体験から始まり，API キーの取得，Python SDK を用いた環境構築とコーディング，さらにはモデルやパラメータの調整までを体験した．

「ChatGPT を使う」という言葉は，日常的には UI を指すが，技術的な文脈では「OpenAI API を通じて GPT-4o を利用する」といった表現がより正確である．同様に，本演習で実施したのは「Gemini API を通じて Gemini モデルを利用する」ことであったと理解することが重要である．この階層構造を理解することは，生成 AI を単なるチャットツールとしてではなく，高度な情報処理部品として応用するための第一歩となる．