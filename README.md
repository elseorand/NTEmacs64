NTEmacs64
=========

Windows版Emacs(通称NTEmacs)の64ビット版 version 24.3.93 (24.4 の pretest 版)

![emacs](app.png)

※ メニューバーとツールバーを非表示にして zenburn のテーマを適用した状態の起動画面です。

バイナリ説明
------------

### 起動方法
**emacs24.3.93.zip** を展開すると **emacs24.3.93/** フォルダが出来るので **emacs24.3.93/bin/runemacs.exe** を実行します。

### 特徴
* ソースには一切手を加えずにビルドしています
* MSYS2 (MSYS の改良版) を使用してビルドしているので Cygwin に依存していません
 * Emacs 上でのパスの扱いなどが自然になります
* 64bit 版です
 * 公式ビルドは今のところ Windows 用の 64bit 版バイナリを提供していません
* gcc に **-Ofast -march=corei7 -mtune=corei7** を付けて最適化ビルドされています
* 画像対応させる為の最低限の DLL を同梱しています (**GIF, PNG, JPEG, TIFF, XPM**)
 * 本来は SVG の表示にも対応可能ですが、依存 DLL(主に GTK+ 関連) が多すぎるので含めていません
* **libxml2, GnuTLS, D-Bus** の DLL も同梱しています
 * elisp で実装されたテキストブラウザ M-x eww も動作確認済みです
   ![emacs](app_eww.png)
 * D-Bus は Linux 環境では便利な機能ですが、Windows 上ではほぼ使い道がありませんが、ビルドが通ったため同梱しています
 * 追加した DLL は全て emacs24.3.93/bin/ 以下にあります (bin/*.dll 以外追加したファイルはありません)

### 注意事項
* NTEmacs24.4 からは NTEmacs24.3 とはフォルダ構成が変わっています
 * 古い DDSKK はフォルダ構成を認識できない問題や 24.4 で使用する場合に不具合があります
  * DDSKK は cvs 版のバージョン 15.1.91 以降を使用してください
* movemail.exe で POP3 が使用出来ません (使ってる人は居ないと思いますが…)
 * Gnus は elisp で POP3 の通信を行うので movemail.exe は必須ではありません
* IMEパッチは適用していないので MS-IME などを使用した日本語入力に問題があります
 * これは公式ビルドであっても同じです
 * ちなみに、標準状態で日本語が全く入力出来ない訳ではありません
  * Ctrl-\ (バックスラッシュ) で日本語入力が可能です (変換効率は悪いですが…コメント入力程度であれば問題ありません)
  * 使い方等は emacs kkc で検索してみてください

ビルド方法
----------

<http://alpha.gnu.org/gnu/emacs/pretest/emacs-24.3.93.tar.xz>
を、ソースに一切手を加えずにビルドします。

### MSYS2 のインストール
<http://sourceforge.net/projects/msys2/>
から **msys2-x86_64-20140704.exe** を取得しインストールします。

### 64ビット環境用のシェルの起動
インストールディレクトリ (c:/msys64) 直下の **mingw64_shell.bat** を起動します。

### ビルド関連パッケージのインストール
    $ pacman -S base-devel
    $ pacman -S mingw-w64-x86_64

    どちらもかなり時間が掛かります

### ビルドとインストール
    $ tar xvJf emacs-24.3.93.tar.xz
    $ cd emacs-24.3.93/

    ソースを展開してディレクトリに移動します

    $ export MSYSTEM=MINGW32

    デフォで MINGW64 になって configure がコケるので騙す為です (32bit ビルドになる訳ではありません)
    これがミソ

    $ cp c:/msys64/mingw64/include/noX/xpm.h c:/msys64/mingw64/include/noX/simx.h c:/msys64/mingw64/include/X11

    XPM をサポートさせる為にヘッダファイルをコピーします (configure を書き換えても可能ですが、ソースはいじらない方針なので…)
    
    $ CFLAGS='-Ofast -march=corei7 -mtune=corei7' ./configure --without-pop

    --without-pop すると movemail.exe で POP3 が使えなくなりますが movemail.exe のビルドがコケるので仕方なく…
    (原因は判明していますが、ソースいじらずに通す方法を思案中)
    最適化オプションは適当に
    ※ --prefix=c:/emacs24.4 と指定した時に site-lisp/ が認識されない不具合が発生したので --prefix は付けません

    $ make bootstrap && make install

    これでおしまい (make install-strip するとなぜか exe が壊れます…原因調査中)
    
    --prefix を付けないと c:/msys64/usr/local/ 以下にインストールされています

### 今後の予定
* サウンドサポート (Windows でもサポート出来るようなので現在調査中、優先度高)
* movemail.exe の POP3 対応 (使ってる人は居ないと思うので、優先度低)
* IME パッチの適用 (基本的にソースには一切手を加えない方針なので、優先度は低)
