# Flaskで作る画像判定

## virtualenv環境作成

バーチャル環境にvirtualenvを使います。

virtualenvが既にインストールされているか確認

```
virtualenv --version
```

### virtualenvのインストール

バージョン確認してまだインストールされてなければ次の手順でインストールします。既にインストールしていたら、次の「今回の作成アプリの環境設定」に進みます。

virtualenvのインストールコマンド

```
sudo pip install virtualenv
```



### 今回作成アプリの環境設定

flask_vggという環境を作るには、以下を実行

```
virtualenv flask_vgg
```



環境に入るのは以下`cd flask_vgg`で移動した後に次のコマンドを実行

ここはよく失敗するので注意。

要は`virtualenv`で作成した環境名のフォルダ内にbinフォルダが作成されます。binフォルダ内にactivateファイルがあるのでそれを実行すれば良いのです。

```python
source bin/activate
```

現在の仮想環境から出る。

```python
deactivate
```

## アプリケーションの初期設定

Flaskの導入

```
pip install Flask
```

### Flaskバージョン確認

Flaskのバージョン確認は次のようにPythonから確認します。

```
$ python
>> import flask
>> flask.__version__
```

'1.1.1'のようにバージョンが確認できます。

今回のFlaskバージョンは 1.1.1 での例です。

### tensorflowインストール

```
pip install tensorflow
```

## kerasインストール

```
import keras
```

### pillowインストール

```
pip install pillow
```

###matplotlibインストール
```
pip install matplotlib
```



### Flaskドキュメント

[Flask1.1系のドキュメントはこちら](https://flask.palletsprojects.com/en/1.1.x/)

## アプリケーション本体の作成

## Hello World!

簡単にHello World!を表示するには次のように記述します。

ファイル名は hello.py としてflask_vggフォルダに保存します。

hello.py

```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'
```

## サーバー起動

これでサーバーを起動してhello.pyの内容を表示することができます。

まずはexportを行います。

```
export FLASK_APP=hello.py
```

続いてサーバーを起動します。

```
flask run
```

\* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)と表示されますのでこのURLでブラウザを開きます。

サーバーを止めるには、表示に記述されている通り、CTRL+C です。

## ファイルのアップロード機能作成

ファイルアップロードファイル upload.py を新規作成します。

uploadの方法はドキュメントの [Patterns for Flask](https://flask.palletsprojects.com/en/1.1.x/patterns/#patterns) から[Uploading Files](https://flask.palletsprojects.com/en/1.1.x/patterns/fileuploads/)を参考にします。

まず、`uploads`というアップロードファイルを保存するフォルダを作成します。

upload.py コードには必要なライブラリをimportしています。

`from werkzeug.utils import secure_filename`は不正なファイルをアップロードできないようにサニタイズするライブラリの読み込みです。

`UPLOAD_FOLDER = '/uploads'`はアップロードフォルダの指定です。

`ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif'}`はアップロードできるファイルの拡張子を決めています。

upload.py 　支所の記述

```
import os
from flask import Flask, flash, request, redirect, url_for
from werkzeug.utils import secure_filename

UPLOAD_FOLDER = '/uploads'
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif'}

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
```

続いてDocumentをさらに参考にします。upload.pyを次のようにコードを追加します。

allowed_file()の定義

```
def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS
```

* `rsplit('.', 1)[1]`は第1引数の値で第2引数の回数だけ分割します。つまり画像ファイルのファイル名と拡張子を分割してindex1のものということになります。その結果拡張子部分の判定になります。
  さらにlower()により拡張子が大文字でも小文字でも小文字に変換されます。
* `x in y`の記述は、`x`が`y`に含まれていると`True`、含まれていないと`False`を返します。

この関数では `in` を使って、ファイル名に` . `が含まれているかどうか（拡張子があるかどうか）と`ALLOWED_EXTENSIONS`の値が含まれているかどうかをチェックして、True、Falseを返す関数を定義しています。

引数には何らかのファイルネームが入るようになります。



続いて、ルートアドレスの挙動をデコレートを使って記述します。

挙動についてはupload_file()で定義します。

```
@app.route('/', methods=['GET', 'POST'])
def upload_file():
     if request.method == 'POST':
        # check if the post request has the file part
        if 'file' not in request.files:
            flash('No file part')
            return redirect(request.url)
        file = request.files['file']
        # if user does not select file, browser also
        # submit an empty part without filename
        if file.filename == '':
            flash('No selected file')
            return redirect(request.url)
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
            return redirect(url_for('uploaded_file',
                                    filename=filename))
    return '''
    <!doctype html>
    <html>
    <head>
    <meta charset="UTF-8">
    <title>Upload new File</title>
    </head>
    <body>
    <h1>Upload new File</h1>
    <form method=post enctype=multipart/form-data>
      <input type=file name=file>
      <input type=submit value=Upload>
    </form>
    </body>
    </html>
    '''
```





upload.py　

```
import os
from flask import Flask, flash, request, redirect, url_for
from werkzeug.utils import secure_filename

UPLOAD_FOLDER = './uploads'
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif'}

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        # check if the post request has the file part
        if 'file' not in request.files:
            flash('No file part')
            return redirect(request.url)
        file = request.files['file']
        # if user does not select file, browser also
        # submit an empty part without filename
        if file.filename == '':
            flash('No selected file')
            return redirect(request.url)
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
            return redirect(url_for('uploaded_file',
                                    filename=filename))
    return '''
    <!doctype html>
    <html>
    <head>
    <meta charset="UTF-8">
    <title>Upload new File</title>
    </head>
    <body>
    <h1>Upload new File</h1>
    <form method=post enctype=multipart/form-data>
      <input type=file name=file>
      <input type=submit value=Upload>
    </form>
    </body>
    </html>
    '''
```

### ファイルアップロード機能の確認

まずはexportを行います。

```
export FLASK_APP=upload.py
```

続いてサーバーを起動します。

```
flask run
```

この段階ではエラーになります。

```
# Internal Server Error

The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.
```

ただし、この段階でもuploadsフォルダに画像はアップロードされていますので確認しておきます。もし画像がなければどこか間違っています。



uploades_file関数を定義します。

```
from flask import send_from_directory

@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'],
                               filename)
```

これで画像がアップロードされるとuploadsフォルダに保存されて、ブラウザにその画像が表示されることになります。



## 画像の予測

```
from keras.modules import Sequential, load_model
import keras.sys
import numpy as numpy
from PIL import Image
```







```
filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)

            model = load_model('./animal_cnn_aug.h5')

            image = Image.open(filepath)
            image = image.convert('RGB')
            image = image.resize((image_size, image_size))
            data = np.asarray(image)
            X = []
            X.append(data)
            X = np.array(X)

            result = model.predict([X])[0]
            predicted = result.argmax()
            percentage = int(result[predicted] * 100)

            return "ラベル： " + classes[predicted] + ", 確率："+ str(percentage) + " %"

```

```
import os
from flask import Flask, flash, request, redirect, url_for
from werkzeug.utils import secure_filename

from keras.modules import Sequential, load_model
import keras.sys
import numpy as numpy
from PIL import Image

UPLOAD_FOLDER = './uploads'
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif'}

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        # check if the post request has the file part
        if 'file' not in request.files:
            flash('No file part')
            return redirect(request.url)
        file = request.files['file']
        # if user does not select file, browser also
        # submit an empty part without filename
        if file.filename == '':
            flash('No selected file')
            return redirect(request.url)
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
            filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)

            model = load_model('./animal_cnn_aug.h5')

            image = Image.open(filepath)
            image = image.convert('RGB')
            image = image.resize((image_size, image_size))
            data = np.asarray(image)
            X = []
            X.append(data)
            X = np.array(X)

            result = model.predict([X])[0]
            predicted = result.argmax()
            percentage = int(result[predicted] * 100)

            return "ラベル： " + classes[predicted] + ", 確率："+ str(percentage) + " %"




            # return redirect(url_for('uploaded_file',
            #                         filename=filename))
    return '''
    <!doctype html>
    <html>
    <head>
    <meta charset="UTF-8">
    <title>Upload new File</title>
    </head>
    <body>
    <h1>Upload new File</h1>
    <form method=post enctype=multipart/form-data>
      <input type=file name=file>
      <input type=submit value=Upload>
    </form>
    </body>
    </html>
    '''
from flask import send_from_directory

@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'],
                               filename)
                               
```

