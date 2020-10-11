# COM3D2へのBepInEx 5.3環境の導入メモ
2020年8月15日ごろ、BepInEx5.3を導入したので、調べたことのメモを残しておく。

# なぜBepInExか

メリット

- プラグインがUnity製ゲーム全般で共用できるので、汎用のプラグインが実行できる。
- 便利プラグイン例
  - [RuntimeUnityEditor](https://github.com/ManlyMarco/RuntimeUnityEditor)
    - ゲーム内でGameObjectツリーを閲覧したり、対話的にC#コードを実行したりできる。
  - [ScriptEngine](https://github.com/BepInEx/BepInEx.Debug)
    - ゲーム実行中にBepInExプラグインを再ロードできる。プラグイン開発時に、ゲームを再起動する必要がなくなるので便利。
  - [ScriptLoader for BepInEx 5](https://github.com/denikson/BepInEx.ScriptLoader)
    - C＃ファイルをDLLにコンパイルせずに実行できるBepInEx 5プラグイン。簡単なMODに便利。

デメリット

- 既存のSybaris2.x用プラグインも共存させて呼出せるが、そのままでは動作しないプラグインは多そう。

  - ↓のようなエラーが延々と出続けて、ゲームが起動できなくなる。

  - ```
    [Info   :   Console] OnLevelWasLoaded was found on （プラグイン名）
    ```
    
  - `OnLevelWasLoaded()`はUnityの新しいバージョンでは廃止されて、`SceneManager.sceneLoaded`へ移行している。よく使うメソッドなので、この問題で起動できないプラグインは多いと思われる。



まとめると、プラグインのユーザは導入すべきでない、開発者ならプラグイン開発におけるメリットがあるので一考の価値あり。



# BepInEx v4.xとv5.xの違い

2020年8月15日時点で、ネットで見つかるBepInExの導入方法はv4とv5の情報が混在してる。

BepInEx v4では「config.ini」だったものが、BepInEx v5では「configフォルダの中の複数の.cfgファイル」に置き換わっている。ネットで情報を探すときは適宜読み替えること。



# 使用するツール

あらかじめダウンロードしておくこと

- BepInEx
  - プロジェクトページ： https://github.com/BepInEx/BepInEx
  - BepInEx 5.3 ダウンロード：https://github.com/BepInEx/BepInEx/releases/tag/v5.3
- BepInEx.UnityInjectorLoader
  - プロジェクトページ：https://github.com/BepInEx/BepInEx.UnityInjectorLoader
  - BepInEx.UnityInjectorLoader 1.3.0.0（ BepInEx 5.0 RC1より新しいバージョンに対応）　ダウンロード：https://github.com/BepInEx/BepInEx.UnityInjectorLoader/releases/tag/v1.3.0.0
- BepInEx.SybarisLoader.Patcher
  - プロジェクトページ：https://github.com/BepInEx/BepInEx.SybarisLoader.Patcher
  - BepInEx.SybarisLoader.Patcher 1.3.0.0（BepInEx 5.0 RC1より新しいバージョンに対応）　ダウンロード：https://github.com/BepInEx/BepInEx.SybarisLoader.Patcher/releases/tag/v1.3.0.0



# 動作確認時のバージョン

- カスタムオーダーメイド３D2　Version 1.45.0 
- BepInEx 5.3.0.0
- BepInEx.UnityInjectorLoader 1.3.0.0
- BepInEx.SybarisLoader.Patcher 1.3.0.0
- Sybaris 2.2



# 導入手順


1. いくつかのSybarisのDLLを削除：以下のDLLファイルで **存在するもの全て** をゲームフォルダ内から削除します
    - `Sybaris\ExIni.dll`
    - `Sybaris\UnityInjector.dll`
    - `Sybaris\Mono.Cecil.dll`
    - `Sybaris\Sybaris.Loader.dll`
    - `Sybaris\COM3D2.UnityInjector.Patcher.dll`
      -  (またはSybaris.UnityInjector.Patcher等の他のUnityInjectorパッチャーを使っている場合)
    - `opengl32.dll`
      - （ゲームのルートフォルダから）
    
2. 「BepInEX」を展開して、ゲームフォルダに配置

3. 「BepInEx.SybarisLoader.Patcher」を展開して、BepInExフォルダーに配置

4. 「BepInEx.UnityInjectorLoader」を展開して、BepInExフォルダーに配置

5. ゲームを起動する
	
	- `BepInEX\config\*.cfg`が作成される
	- この時点ではゲームは正常に起動できて、スタート画面が表示されるはず。
	
6. `BepInEX\config\BepInEx.cfg`の以下の箇所を`Enabled = true`へ変更

   - ```
     ## Enables showing a console for log output.
     # Setting type: Boolean
     # Default value: false
     Enabled = false
     ```
   - Sybarisを導入したときのように、ゲーム起動時にゲーム画面の他に黒いコンソールウィンドウも表示されるようになる。
   
7. `BepInEx\config\org.bepinex.plugins.unityinjectorloader.cfg`の以下の箇所を`UnityInjector = Sybaris\UnityInjector`へ変更

   - ```
     ## Location of UnityInjector folder relative to game root
     ## Can be an absolute path
     # Setting type: String
     # Default value: UnityInjector
     UnityInjector = UnityInjector
     ```

   - Sybaris2.x用プラグインの配置されているフォルダを指定することで、Sybaris系プラグインが読み込まれるようになる。

8. ゲームを起動する。ゲームが起動できずスタート画面が表示されないなら、以下の対応をする。

    - 自分で確認したときは、「GameMain::LoadScene SceneWarningoadScene SceneWarning」のようなエラーでゲームが起動できなかった。

## ゲームが起動できないときの対応

1. UnityInjectorフォルダのdllをすべて除外して、ゲームが起動できるか確認。
	- ゲームが起動できれば、問題は除外したプラグインdllのいずれかということになる。
2. 除外したUnityInjectorフォルダのdllを順番に追加して、起動できない原因のdllを探す。   　



# 参考ページ
- BepInEX　公式ページ
  - [Installing BepInEx](https://bepinex.github.io/bepinex_docs/master/articles/user_guide/installation/index.html?tabs=tabid-win)
    - BepInEX 5系のインストール方法。
  - [Upgrading#Migrating from Sybaris 2.x](https://bepinex.github.io/bepinex_docs/master/articles/user_guide/upgrading.html#migrating-from-sybaris-2x)
    - Sybaris2.2からBepInEX 5系へのアップグレード方法
  - [How to: create a BepInEx plugin](https://bepinex.github.io/bepinex_docs/master/articles/dev_guide/plugin_tutorial/index.html)
    - BepInExプラグインの基本的な作成方法
  - [List of useful development plugins](https://bepinex.github.io/bepinex_docs/master/articles/dev_guide/dev_tools.html)
    - プラグイン作成に便利なツールの紹介。ScriptEngineやScriptLoaderなど。
  - [BepInEx 4系と5系の重要な変更点](https://github.com/BepInEx/BepInEx/releases#:~:text=BepInEx%205.0%20RC1)
  - ~~wiki~~  wikiはBepInEx 4系の情報なので注意
    - ~~Sybaris 2.xからBepInEx 4系への移行方法　https://github.com/BepInEx/BepInEx/wiki/Migration_jp~~
    - ~~インストール方法：https://github.com/BepInEx/BepInEx/wiki/Installation_jp#%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB~~
- BepInEX 5.0RCの環境構築時のフォルダ構成と、`*.cfg`ファイルの設定方法
  - https://twitter.com/SuzushiAmaaji/status/1191200391515394048
- BepInEx 4系 とSybaris環境を共存させたときのファイル構成のメモ　http://www17.plala.or.jp/NotPriest008/sybaris.html
- COM3D2.BepInEx.AIO：https://github.com/NeighTools/COM3D2.BepInEx.AIO
  - Sybaris2からBepInEx v4へ移行するために必要なもの一式（SybarisLoader, UnityInjectorLoader含む）が入っている。
  - 「exeファイルを実行する必要がある・BepInEx 4用である」ということで、今回は使わなかった。



# フォルダ構成のツリー

↓の構成例では、上記の必須dll以外にRuntimeUnityEditorを導入した場合。
太字は重要なファイル・フォルダ。

```

BepInEx
│  LogOutput.log
│
├─config
│      **BepInEx.cfg**
│      org.bepinex.plugins.unityinjectorloader.cfg
│      RuntimeUnityEditor.cfg
│      **SybarisLoader.cfg**
│
├─core
│      0Harmony.dll
│      0Harmony.xml
│      BepInEx.dll
│      BepInEx.Harmony.dll
│      BepInEx.Harmony.xml
│      BepInEx.Preloader.dll
│      BepInEx.Preloader.xml
│      BepInEx.xml
│      Mono.Cecil.dll
│      Mono.Cecil.Mdb.dll
│      Mono.Cecil.Pdb.dll
│      Mono.Cecil.Rocks.dll
│      MonoMod.RuntimeDetour.dll
│      MonoMod.RuntimeDetour.xml
│      MonoMod.Utils.dll
│      MonoMod.Utils.xml
│
├─patchers
│      **BepInEx.SybarisLoader.Patcher.dll**
│
└─plugins
    ├─RuntimeUnityEditor
    │      LICENSE
    │      README.md
    │      RuntimeUnityEditor.Bepin5.dll
    │      RuntimeUnityEditor.Core.dll
    │
    └─**UnityInjectorLoader**
            BepInEx.UnityInjectorLoader.dll
            ExIni.dll
Sybaris
│  Mono.Cecil.Inject.dll
│  Mono.Cecil.Rocks.dll
│  （その他、ReiPatcherレス化プラグイン一式で導入した`COM3D2.DistortCorrect.Managed.dll`など）
│
├─UnityInjector
│      **（Sybaris2.x用プラグインのdll）**
└─（その他、Configなどのフォルダ）

```
