NTEmacs64
=========

Windows 版 Emacs (通称 NTEmacs) の 64bit 版 version 25.2

![emacs](app.png)

メニューバーとツールバーを非表示にして zenburn のテーマを適用した状態の起動画面です。


バイナリ説明
------------

公式ビルドに 64bit 版のビルドが作成された為、もはや 64bit 版を独自ビルドする事に意味がなくなりましたが  
公式ビルドを使ってみて幾つか不満があったのと、相変わらず日本語入力に問題を残したままなので  
IMEパッチ版をビルドする事にしました。  
公式ビルドを使用するには以下から取得出来ます。  
<http://ftpmirror.gnu.org/emacs/windows/>

ファイル | 説明
------------- | -------------
[**emacs-25.2-IME-patched.zip**](https://github.com/chuntaro/NTEmacs64/raw/master/emacs-25.2-IME-patched.zip) | IMEパッチ適用版

### 起動方法
**emacs-25.2-IME-patched.zip** を展開すると **emacs-25.2/** フォルダが出来るので **emacs-25.2/bin/runemacs.exe** を実行します。

### 特徴
* MSYS2 (MSYS の改良版) を使用してビルドしているので Cygwin に依存していません
  * Emacs 上でのパスの扱いなどが自然になります
  * 公式ビルドでも MSYS2 を使用しています (ビルド手順も普通にビルドするだけなので、ほぼ一緒と思われます)
  * MSYS2 でビルドはしていますが、実際に使用されているコンパイラツールチェインは MinGW-w64(mingw64) になります
* gcc に **-Ofast -march=x86-64 -mtune=corei7** を付けて最適化ビルドされています
* バージョン 25.2 から追加された目玉機能のダイナミックモジュールを有効にしてビルドされています
  * 公式ビルドはダイナミックモジュールを有効にしてビルドされていません
  * ダイナミックモジュールは DLL に実装された機能を Emacs から直接使用する為の機能ですが、load-path 上に DLL を配置して、なおかつ DLL に専用のエントリーポイントが必要なので任意の DLL がロード可能になるわけではありません
  * [Introduction to Emacs modules](http://diobla.info/blog-archive/modules-tut.html) ← このサイトを参考にモジュール(DLL)を作成して動作確認済みです
* 画像対応させる為の最低限の DLL を同梱しています (**GIF, PNG, JPEG, TIFF, XPM**)
  * 本来は SVG の表示にも対応可能ですが、依存 DLL(主に GTK+ 関連) が多すぎるので含めていません
* **libxml2, GnuTLS** の DLL も同梱しています
  * elisp で実装されたテキストブラウザ M-x eww も動作確認済みです
   ![emacs](app_eww.png)
  * 追加した DLL は全て emacs-25.2/bin/ 以下にあります (bin/*.dll 以外追加したファイルはありません)
* IMEパッチを適用してビルドされています
  * <https://gist.github.com/rzl24ozi> で最小構成に整理されたIMEパッチがアップされたので、それを適用しました  
パッチを整理していただきありがとうございます！  
IMEを有効にするには以下の設定が必要です  
  ```emacs-lisp
    ;; (set-language-environment "UTF-8") ;; UTF-8 でも問題ないので適宜コメントアウトしてください
    (setq default-input-method "W32-IME")
    (setq-default w32-ime-mode-line-state-indicator "[--]")
    (setq w32-ime-mode-line-state-indicator-list '("[--]" "[あ]" "[--]"))
    (w32-ime-initialize)
    ;; 日本語入力時にカーソルの色を変える設定 (色は適宜変えてください)
    (add-hook 'w32-ime-on-hook '(lambda () (set-cursor-color "coral4")))
    (add-hook 'w32-ime-off-hook '(lambda () (set-cursor-color "black")))

    ;; 以下はお好みで設定してください
    ;; 全てバッファ内で日本語入力中に特定のコマンドを実行した際の日本語入力無効化処理です
    ;; もっと良い設定方法がありましたら issue などあげてもらえると助かります
  
    ;; ミニバッファに移動した際は最初に日本語入力が無効な状態にする
    (add-hook 'minibuffer-setup-hook 'deactivate-input-method)
  
    ;; isearch に移行した際に日本語入力を無効にする
    (add-hook 'isearch-mode-hook '(lambda ()
                                    (deactivate-input-method)
                                    (setq w32-ime-composition-window (minibuffer-window))))
    (add-hook 'isearch-mode-end-hook '(lambda () (setq w32-ime-composition-window nil)))
  
    ;; helm 使用中に日本語入力を無効にする
    (advice-add 'helm :around '(lambda (orig-fun &rest args)
                                 (let ((select-window-functions nil)
                                       (w32-ime-composition-window (minibuffer-window)))
                                   (deactivate-input-method)
                                   (apply orig-fun args))))
  ```

### 注意事項
* DDSKK はバージョン 16.1 以降を使用してください
* ダイナミックモジュールの機能は、一旦有効にしてビルドすると設定ファイルで無効にする事が出来ません
  * load-path 上に DLL があるか気になる方は以下のコードを実行すると確認出来ます (あると警告が表示されます)
  ```emacs-lisp
    (dolist (dir load-path)
      (dolist (dll (directory-files dir t "\\.dll$"))
        (warn dll)))
  ```
* 言語の詳細設定の「アプリウィンドウごとに異なる入力方式を設定する」にチェックを入れていると日本語入力に切り替わらない問題があるようです  
詳細は以下の issue を参照してください  
https://github.com/chuntaro/NTEmacs64/issues/3

ビルド方法
----------

### ビルド環境

|エディション|バージョン|OS ビルド|
|---|---|---|
|Windows 10|1607|14393.1066|

|PC 名|アカウント名|
|---|---|
|NTEmacs64|NTEmacs64|

PC 名(ホスト名)とアカウント名(ユーザー名)は emacs.exe に含まれるのでアップロードする場合は適切なものにしておきます。

### MSYS2 のインストール
<http://msys2.github.io/>
から **msys2-x86_64-20161025.exe** (2017/4/26 時点の最新) を取得しインストールします。

### 64ビット環境用のシェルの起動
~~起動用のバッチファイルを用意します。~~ (※ 最新の MSYS2 であればスタートメニューから普通に起動しても PATH に余計なものが含まれなくなったので、下記のバッチファイルは不要になりました)
```bat
 @echo off
 
 set PATH=
 
 call c:\msys64\mingw64.exe
```

スタートメニューの **MSYS2 MinGW 64-bit** で MSYS2 のシェルを起動後、上記サイトに記載されているように MSYS2 を最新にしておきます。

    $ pacman -Syu

    MSYS2 のシェルを再起動します
    上記コマンドを初めて実行すると、一旦終了してもう一度実行するように促されるので、
    右上の×をクリックしてシェルを終了後、もう一度 $ pacman -Syu を実行します

    $ pacman -Su

### ビルド関連パッケージのインストール
    http://git.savannah.gnu.org/cgit/emacs.git/tree/nt/INSTALL.W64
    に書いてあるように以下のコマンドでビルド関連のパッケージをインストールします

    $ pacman -S base-devel \
      mingw-w64-x86_64-toolchain \
      mingw-w64-x86_64-xpm-nox \
      mingw-w64-x86_64-libtiff \
      mingw-w64-x86_64-giflib \
      mingw-w64-x86_64-libpng \
      mingw-w64-x86_64-libjpeg-turbo \
      mingw-w64-x86_64-librsvg \
      mingw-w64-x86_64-libxml2 \
      mingw-w64-x86_64-gnutls \
      mingw-w64-x86_64-zlib

### ソースの取得と検証
    $ wget http://ftpmirror.gnu.org/emacs/emacs-25.2.tar.xz
    $ wget http://ftpmirror.gnu.org/emacs/emacs-25.2.tar.xz.sig
    $ wget http://ftp.gnu.org/gnu/gnu-keyring.gpg

    Emacs のソースと GNU 関連の公開鍵を取得し、ソースを検証します

    $ gpg --verify --keyring ./gnu-keyring.gpg emacs-25.2.tar.xz.sig
    gpg: 署名されたデータが'emacs-25.2.tar.xz'にあると想定します
    gpg: 2017年04月22日 00時33分38秒 JSTにRSA鍵ID 7C207910で施された署名
    gpg: "Nicolas Petton <nicolas@petton.fr>"からの正しい署名
    gpg:                 別名"Nicolas Petton <petton.nicolas@gmail.com>"
    gpg:                 別名"Nicolas Petton <nicolas@foretagsplatsen.se>"
    gpg: *警告*: この鍵は信用できる署名で証明されていません!
    gpg:       この署名が所有者のものかどうかの検証手段がありません。
    主鍵フィンガー・プリント: 28D3 BED8 51FD F3AB 57FE  F93C 2335 87A4 7C20 7910

    https://www.gnu.org/software/emacs/download.html
    上記サイトの中ほどにフィンガー・プリントが記載されているので確認します

    https://gist.github.com/rzl24ozi
    上記サイトよりIMEパッチ emacs-25.2-w32-ime.diff を取得します

### ビルドとインストール
    $ tar xvJf emacs-25.2.tar.xz
    $ cd emacs-25.2/

    ソースを展開してディレクトリに移動します

    Emacs はビルドしたディレクトリのフルパスを emacs.exe に含めて記憶し
    *Help* から C のソースへ飛ぶ場合にそのパスを使います。(もちろん init.el で変更可能)
    なので、ソースは c:\ の直下に展開した方がいいです。(個人的なディレクトリ名を含めない為でもあります)

    $ patch.exe -p0 < ../emacs-25.2-w32-ime.diff
    $ autoconf

    IMEパッチを適用後 configure スクリプトを更新します

    $ CFLAGS='-Ofast -march=x86-64 -mtune=corei7 -static' ./configure --prefix=c:/emacs-25.2 \
      --without-dbus --without-compress-install --with-modules

    configure を実行します
    その際に CFLAGS で最適化オプションを指定します
    -static は公式ビルドで指定されていたので同じように指定します

    configure のオプションは --with-modules 以外は公式ビルドで指定されていたので同じように指定します
    --with-modules でダイナミックモジュールが有効になります

    25.2 から --prefix を付けないと c:/msys64/mingw64/ 以下にインストールされてしまうので --prefix を付けます
    インストール先は適宜変更してください

    ..途中省略して、以下 configure の結果です
    
    Configured for 'x86_64-w64-mingw32'.
    
      Where should the build process find the source code?    .
      What compiler should emacs be built with?               gcc  -Ofast -march=x86-64 -mtune=corei7 -static
      Should Emacs use the GNU version of malloc?             no
        (The GNU allocators don't work with this system configuration.)
      Should Emacs use a relocating allocator for buffers?    no
      Should Emacs use mmap(2) for buffer allocation?         yes
      What window system should Emacs use?                    w32
      What toolkit should Emacs use?                          none
      Where do we find X Windows header files?                NONE
      Where do we find X Windows libraries?                   NONE
      Does Emacs use -lXaw3d?                                 no
      Does Emacs use -lXpm?                                   yes
      Does Emacs use -ljpeg?                                  yes
      Does Emacs use -ltiff?                                  yes
      Does Emacs use a gif library?                           yes
      Does Emacs use a png library?                           yes
      Does Emacs use -lrsvg-2?                                yes
      Does Emacs use cairo?                                   no
      Does Emacs use imagemagick?                             no
      Does Emacs support sound?                               yes
      Does Emacs use -lgpm?                                   no
      Does Emacs use -ldbus?                                  no
      Does Emacs use -lgconf?                                 no
      Does Emacs use GSettings?                               no
      Does Emacs use a file notification library?             yes (w32)
      Does Emacs use access control lists?                    yes
      Does Emacs use -lselinux?                               no
      Does Emacs use -lgnutls?                                yes
      Does Emacs use -lxml2?                                  yes
      Does Emacs use -lfreetype?                              no
      Does Emacs use -lm17n-flt?                              no
      Does Emacs use -lotf?                                   no
      Does Emacs use -lxft?                                   no
      Does Emacs directly use zlib?                           yes
      Does Emacs have dynamic modules support?                yes
      Does Emacs use toolkit scroll bars?                     yes
      Does Emacs support Xwidgets (requires gtk3)?            no
    
      Does Emacs support W32-IME?                             yes
      Does Emacs support RECONVERSION?                        yes
      Does Emacs support DOCUMENTFEED?                        yes

    $ make bootstrap && make install-strip

    終了後 --prefix で指定したディレクトリにインストールされています

## 謝辞
そもそもの発端は、公式ビルド含め巷のビルドは自分で納得できるものがなかったので  
色々調べているうちに MSYS2 を使えば出来そうな事が分かり、ビルドしてみた事が始まりでした。  

最終的に「64bit版・最適化ビルド・ソース未変更」のビルドを作成する事は出来たので、ひとまず満足しています。  
さらに [rzl24ozi 氏](https://gist.github.com/rzl24ozi) によりIMEパッチを整理して頂いた為IMEパッチ版も追加してみました。  

最後に、このような素晴らしいソフトウェアを生み出し改良し続けている関係者の方々に最大限の感謝を表し──  

Have fun!



ビルド関連追記
--------------

### emacs-25.2/bin/*.dll の依存関係など
* 以下の DLL 以外追加したファイルはありません
* DLL は全て MSYS2 からコピーしたものです
* 依存関係は Windows に標準インストールされているものは含めていません
```
 XPM
 libXpm-noX4.dll
 
 JPEG
 libjpeg-8.dll
 
 PNG
 libpng16-16.dll
 └ zlib1.dll
 
 GIF
 libgif-7.dll
 
 TIFF
 libtiff-5.dll
 ├ libjpeg-8.dll
 ├ liblzma-5.dll
 └ zlib1.dll
 
 LIBXML2
 libxml2-2.dll
 ├ libiconv-2.dll
 ├ liblzma-5.dll
 └ zlib1.dll
 
 GnuTLS
 libgnutls-30.dll
 ├ libwinpthread-1.dll
 ├ libgmp-10.dll
 ├ libhogweed-4.dll
 │ ├ libgmp-10.dll
 │ └ libnettle-6.dll
 ├ libidn-11.dll
 │ ├ libiconv-2.dll
 │ └ libintl-8.dll
 ├ libintl-8.dll
 │ └ libiconv-2.dll
 ├ libnettle-6.dll
 ├ libp11-kit-0.dll
 │ ├ libffi-6.dll
 │ └ libintl-8.dll
 ├ libtasn1-6.dll
 ├ libunistring-2.dll
 │ └ libiconv-2.dll
 └ zlib1.dll
```

### Help から C のソースに設定無しで飛ぶ方法
* emacs-25.2-IME-patched.zip を c:/emacs-25.2 に展開して emacs-25.2.tar.xz を同じフォルダに展開すると
Help から C のソースに自動で飛ぶようになります
(emacs-25.2-IME-patched.zip と emacs-25.2.tar.xz は被るフォルダやファイルは無いので上書きの心配はありません)
 * c:/emacs-25.2 でないと毎回聞かれる事になるので、以下の設定が必要です
  ```emacs-lisp
    (setq source-directory "/path/to/emacs/source/dir")
  ```
 * ソースを全部コピーすると *Help* 参照時に妙なエラーになる場合は、ひとまず src フォルダのみコピーするとよさそうです
