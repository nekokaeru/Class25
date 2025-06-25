### **Gemini API 入門：Python による生成 AI アプリケーション開発体験**

#### **1. はじめに**

近年，生成 AI (Artificial Intelligence, AI) が注目を集めている．しかし，一般に「ChatGPT」という名称が指すものは，ユーザーが対話するためのユーザーインターフェース (User Interface, UI) であり，その背後で動作する大規模言語モデル (Large Language Model, LLM) そのものではない．例えば OpenAI 社が提供するサービスでは，ChatGPT という UI を通じて，GPT-4o や o3 といった LLM を利用する．

この関係は，Google 社が提供する生成 AI でも同様である．Web ブラウザ上で利用できる Google AI Studio は UI であり，その内部では Gemini と呼ばれる LLM が動作している．

本演習では，生成 AI の基本的な構造を理解するため，Google の Gemini を題材として，プログラムから LLM を直接利用する方法を習得する．具体的には，Python というプログラミング言語と，Gemini を操作するためのソフトウェア開発キット (Software Development Kit, SDK) を用いて，簡易的なアプリケーション開発を体験する．

> **【重要】API の進化と公式情報の確認について**
> 生成 AI の技術は非常に速いスピードで進化しており，Google の API も頻繁に更新される．そのため，Web 上のチュートリアルや古い資料に記載されているコードは，すぐに利用できなくなる可能性がある．
> **もしプログラムの実行中にエラーが発生したり，予期せぬ挙動を示したりした場合は，まず最初に[公式の Google Gen AI SDK ドキュメント](https://googleapis.github.io/python-genai/)を確認する必要がある．** 本資料自体も，SDK の重要な変更（`GenerativeModel` から `Client` への移行など）を反映して改訂されている．このことからも，公式情報を正とすることの重要性がわかる．

### **2. 演習の構成**

本演習は以下の手順で実施する．

1.  **Google AI Studio の利用**: まずは UI を通じて Gemini の基本的な性能を体験する．
2.  **API キーの取得**: プログラムから Gemini を利用するために必要な認証キーを取得する．
3.  **開発環境の準備**: Python プログラムを実行するための環境を構築する．
4.  **Python による API の利用**: 実際にコードを記述し，Gemini との対話を自動化する．
5.  **応用**: モデルの性能や挙動を変化させるパラメータを調整する．

---

### **3. 演習 1：Google AI Studio によるモデル挙動の把握**

はじめに，UI である Google AI Studio を利用して，Gemini がどのようなものかを直感的に理解する．

1.  Web ブラウザで Google AI Studio ([https://aistudio.google.com/](https://aistudio.google.com/)) にアクセスし，自身の Google アカウントでログインする．
2.  チャット形式の入力画面でプロンプト（指示文）を入力し，AI との対話を試みる．
3.  画面右上では，対話に使用するモデルを選択できる．高性能な「Gemini 2.5 Pro」や高速・軽量な「Gemini 1.5 Flash」の違いを体感せよ．なお，これらのモデル名は UI 上での表示名であり，プログラムから利用する際は後述する `gemini-1.5-pro-latest` のようなモデル ID で指定する．

##### **3.1 AI Studio の学習補助ツールとしての活用**

本演習を進める中で，不明な専門用語（例：API, SDK）やプログラミングの概念が出現した場合，この AI Studio に直接質問することが推奨される．AI Studio は，本演習における強力な学習アシスタントとなり得る．

---

### **4. 演習 2：Python による Gemini API の利用**

次に，UI を介さず，Python プログラムから直接 Gemini を呼び出す方法を学ぶ．このプログラムと LLM の仲立ちをする仕組みを，アプリケーションプログラミングインターフェース (Application Programming Interface, API) と呼ぶ．

##### **4.1 API キーの取得**

1.  Google AI Studio の画面左側にあるメニューから「**Get API key**」を選択する．
2.  「**Create API key**」ボタンをクリックし，API キーを生成する．
3.  `AIza...` から始まる文字列が API キーである．このキーは他者に知られないよう，安全な場所に保管する必要がある．

##### **4.2 開発環境の準備**

本演習では，Web ブラウザ上で完結する Google Colaboratory (Colab) の利用を推奨する．

**A) Google Colaboratory を利用する場合 (推奨)**

1.  Google Colab ([https://colab.research.google.com/](https://colab.research.google.com/)) にアクセスし，「ノートブックを新規作成」する．
2.  ライブラリをインストールまたはアップグレードする．以下のコマンドをコードセルに入力し，実行する．`-U` オプションにより，常に最新版が保証される．
    ```bash
    !pip install -q -U google-genai
    ```
3.  API キーを設定する．Colab 画面左側の**鍵アイコン (シークレット)** をクリックし，「新しいシークレットを追加」を選択する．**名前**に `GEMINI_API_KEY`，**値**にコピーした API キーを入力し，トグルをオンにする．

**B) ローカル環境を利用する場合**

自身の PC に Anaconda 等が導入されている場合は，以下の手順で環境を構築する．
1.  ターミナル（Windows の場合は Anaconda Prompt や PowerShell）を開き，ライブラリをインストールする．
    ```bash
    pip install -U google-genai
    ```
2.  API キーを環境変数として設定する．この設定方法は OS やシェルによって異なる．（`YOUR_API_KEY_HERE` の部分を自身のキーに置き換えること）
    *   **macOS/Linux (Bash/Zsh)**: `export GEMINI_API_KEY='YOUR_API_KEY_HERE'`
    *   **Windows (PowerShell)**: `$Env:GEMINI_API_KEY="YOUR_API_KEY_HERE"`
    *   **Windows (コマンドプロンプト)**: `setx GEMINI_API_KEY "YOUR_API_KEY_HERE"`

##### **4.3 動作確認コードの実行**

環境が整ったら，最新の推奨方式である **`Client` オブジェクト** を用いて API を呼び出す．以下のコードは，堅牢なキー読み込みと，より具体的なエラーハンドリングを実装している．

```python
import os
from google import genai

api_key = None
# 1. Colab Secrets からキー取得を試行
#    Colab環境でない場合は ImportError が発生し，キーが未登録の場合は
#    userdata.get() が例外を投げる可能性がある．
try:
    from google.colab import userdata
    api_key = userdata.get('GEMINI_API_KEY')
except Exception:
    pass

# 2. Colab Secrets にキーがなかった場合，またはColabでない場合に環境変数をチェック
if not api_key:
    api_key = os.getenv('GEMINI_API_KEY')

# 3. 最終的にキーが見つからない場合にエラーを発生させる
if not api_key:
    raise ValueError("APIキーが見つからない．Colab Secretsまたは環境変数に設定すること．")

try:
    # Clientオブジェクトの初期化
    client = genai.Client(api_key=api_key)

    # Geminiの呼び出し
    response = client.models.generate_content(
        model="gemini-1.5-flash-latest",
        contents="日本の首都について，簡潔に説明せよ．"
    )
    print(response.text)

# より具体的なエラーハンドリング
# genai.errors は genai モジュールを通じてアクセス可能
except genai.errors.PermissionDenied as e:
    print(f"認証エラー: APIキーが不正か，APIが有効化されていない可能性がある．\n詳細: {e}")
except genai.errors.ResourceExhausted as e:
    print(f"レート制限エラー: 無料利用枠の上限に達した可能性がある．時間をおいて再試行すること．\n詳細: {e}")
except Exception as e:
    print(f"予期せぬエラーが発生した: {e}")
```
実行後，「日本の首都は東京である．...」といった応答が表示されれば成功である．例えは，"日本の首都について，カエルの精霊娘っぽく説明せよ．" 等条件を変更して試してみること．

---

### **5. 応用演習：モデルとパラメータの調整**

##### **5.1 モデルの切り替え**

Gemini API では，用途やコストに応じて複数のモデルを使い分けることが可能である．

*   **`gemini-2.5-pro`**: 性能を最も重視する場合の最上位モデル．複雑な推論や質の高い文章生成に適する．
*   **`gemini-1.5-flash-latest`**: 速度と効率に特化した軽量モデル．応答が非常に速いため，チャットボットのようなリアルタイム性が重要なアプリに向いている．
*   **`gemma-3n-e4b-it`: Gemma 3n シリーズのモバイル向け軽量モデル.

モデルを切り替えるには，`model="..."` の ID を書き換えるだけでよい．

```python
# 高性能モデル (Pro) を利用する場合
# response = client.models.generate_content(
#     model="gemini-2.5-pro",
#     contents="50語程度の英文を自然な日本語に翻訳せよ．"
# )

# 高速モデル (Flash) を利用する場合
response = client.models.generate_content(
    model="gemini-1.5-flash-latest",
    contents="50語程度の英文を自然な日本語に翻訳せよ．"
)
print(response.text)
```

##### **5.2 パラメータの調整**

LLM の応答は，`GenerationConfig` で指定するパラメータによって調整できる．

```python
# 応答の多様性や一貫性を制御するパラメータ群
generation_config = genai.GenerationConfig(
    temperature=0.9,
    top_p=0.95,
    top_k=40,
    max_output_tokens=200
)

response = client.models.generate_content(
    model="gemini-1.5-flash-latest",
    contents="面白い物語の冒頭を考えよ．",
    generation_config=generation_config
)
print(response.text)
```

*   **`temperature`**: 応答のランダム性．1 に近いほど多様で創造的になる．
*   **`top_k` / `top_p`**: 次のトークン候補を絞り込む手法．不自然なトークンの出現を抑制する．
*   **`max_output_tokens`**: 生成されるテキストの最大長．

創造的な文章なら `temperature` を高めに，事実に基づいた応答なら低めに設定するなど，目的に応じた調整が有効である．

### **6. まとめ**

本演習を通じて，UI と LLM の違いを理解し，Python から API 経由で LLM を利用する基本を学んだ．具体的には，Google AI Studio での体験，API キーの取得，最新の SDK (`google-genai` の `Client`) を用いたプログラミング，そしてモデルやパラメータの調整を体験した．

この「API を通じてモデルを利用する」という階層構造を理解することは，生成 AI を単なるチャットツールとしてではなく，高度な情報処理部品として応用するための第一歩となる．
