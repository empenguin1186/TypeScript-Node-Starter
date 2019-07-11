# 概要
この資料は README.md を読んでまとめた資料である

## TypeScript + Node
### Project Structure

ファイル構成は以下の通り
```Shell
$ pwd
<local-path>/TypeScript-Node-Starter
$ ls
LICENSE                 dist                    package-lock.json       tsconfig.json
README.md               jest.config.js          package.json            tslint.json
copyStaticAssets.ts     memo.md                 src                     views
debug.log               node_modules            test
```

ここで`src`フォルダは.tsファイルを保存する場所、`dist`フォルダはコンパイルで生成された.jsファイルを保存する場所である。

### Configuring TypeScript compilation

sourceMapとは?
デバッグ時にコンパイルで生成されたjsファイルとコンパイル元のtsファイルをマッピングするファイルを指す
https://developer.mozilla.org/ja/docs/Tools/Debugger/How_to/Use_a_source_map


### Running the build
ビルドコマンドは`npm`コマンドを使用する。なお、package.jsonの`script`タグで実行するビルドコマンドのエイリアスを設定することが可能。

```JSON
"scripts": {
    "start": "npm run serve",
    "build": "npm run build-sass && npm run build-ts && npm run tslint && npm run copy-static-assets",
    "serve": "node dist/server.js",
    "watch-node": "nodemon dist/server.js",
    "watch": "concurrently -k -p \"[{name}]\" -n \"Sass,TypeScript,Node\" -c \"yellow.bold,cyan.bold,green.bold\" \"npm run watch-sass\" \"npm run watch-ts\" \"npm run watch-node\"",
    "test": "jest --forceExit --coverage --verbose",
    "watch-test": "npm run test -- --watchAll",
    "build-ts": "tsc",
    "watch-ts": "tsc -w",
    "build-sass": "node-sass src/public/css/main.scss dist/public/css/main.css",
    "watch-sass": "node-sass -w src/public/css/main.scss dist/public/css/main.css",
    "tslint": "tslint -c tslint.json -p tsconfig.json",
    "copy-static-assets": "ts-node copyStaticAssets.ts",
    "debug": "npm run build && npm run watch-debug",
    "serve-debug": "nodemon --inspect dist/server.js",
    "watch-debug": "concurrently -k -p \"[{name}]\" -n \"Sass,TypeScript,Node\" -c \"yellow.bold,cyan.bold,green.bold\" \"npm run watch-sass\" \"npm run watch-ts\" \"npm run serve-debug\""
  },
```

例えば、アプリを実行する場合は`npm start`を実行すれば良い

### Type Definition (.d.ts) Files
#### .d.tsファイルとは?
型定義ファイル。型定義がされていないjsファイルを使用する時に用いられる。
外部ライブラリの型定義は ./package.json の`devDependencies`フィールドで指定することができる

```JSON
"devDependencies": {
    "@types/async": "^3.0.0",
    "@types/bcrypt-nodejs": "^0.0.30",
    "@types/bluebird": "^3.5.27",
    ...
```

`npm install`を実行すると型定義ファイルは `./node_modules/@types` 配下に配置される

例
```Shell
ls -l node_modules/@types/async/
total 56
-rw-r--r--  1 ...   1183  6  5 05:54 LICENSE
-rw-r--r--  1 ...    757  6  5 05:54 README.md
-rw-r--r--  1 ...  15765  6  5 05:54 index.d.ts // これが型定義ファイル
-rw-r--r--  1 ...   2148  7  8 08:27 package.json
```

node_modules/@types/async/index.d.ts の内容
```TypeScript
...
export interface AsyncQueue<T> {
    length(): number;
    started: boolean;
    running(): number;
    idle(): boolean;
    concurrency: number;
    push<R, E = Error>(task: T | T[], callback?: AsyncResultCallback<R, E>): void;
    unshift<E = Error>(task: T | T[], callback?: ErrorCallback<E>): void;
    remove(filter: (node: DataContainer<T>) => boolean): void;

    saturated(): Promise<void>;
    saturated(handler: () => void): void ;
    empty(): Promise<void>;
    empty(handler: () => void): void;
    drain(): Promise<void>;
    drain(handler: () => void): void;

    paused: boolean;
    pause(): void;
    resume(): void;
    kill(): void;
    workersList<TWorker extends DataContainer<T>, CallbackContainer>(): TWorker[];
    error(error: Error, data: any): void;
    unsaturated(): void;
    buffer: number;
}
...
```

### What if a library isn't on DefinitelyType
特定のライブラリにおいて node_modules/@types 配下に `d.ts` ファイルが存在しない場合は自分で`src` 配下に `d.ts` ファイルを作成する必要がある。

#### Setting up TypeScript to look for .d.ts files in another folder
`src` 配下に `d.ts` ファイルを作成したのちは `tsconfig.json` にパスを追記する必要がある。

```JSON
"baseUrl": ".",
"paths": {
    "*": [
        "node_modules/*",
        "src/types/*"
    ]
}
```
これで自作の `d.ts` ファイルが読み込まれるようになる。コンパイラ時の挙動としては、まず `node_modules/@types` 配下に`d.ts` ファイルが存在するかどうかを確認したのち、見つからなかった場合は `src/types/` 配下に`d.ts` ファイルが存在するかどうかを確認する。

#### Using dts-gen

`dts-gen` の参考文献
https://www.npmjs.com/package/dts-gen

```
dts-gen is a tool that generates TypeScript definition files (.d.ts) from any JavaScript object.
```

#### Writing a .d.ts file
とりあえず自作した`.d.ts` ファイルを使用して外部モジュールを読み込みたい場合はファイル内で以下を記述する。

```TypeScript
declare module "<some-library>";
```
安全なアプリケーションを作成するために`.d.ts` ファイルの作成について学びたい場合は以下を参照
http://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html

### Summary of .d.ts management
`.d.ts` ファイルに関するベストプラクティスは以下の通り。

1. パッケージをインストールする場合に `.d.ts` ファイルを `@types` 経由でインストールできるかを試す
2. パッケージに`.d.ts` ファイルが存在した場合は終了、しなかった場合は、3.に進む
3. `.d.ts` ファイルを自作する
4. `dts-gen` コマンドで`.d.ts` ファイルを自動生成する
5. 失敗した場合は`types`フォルダ配下に<some-library>.d.tsファイルを生成する
6. 作成したファイルに以下を追記
```
declare module "<some-library>";
```
ここまでくればコンパイルは通るはず

### Using the debugger in VS Code
https://qiita.com/TsuyoshiUshio@github/items/9879ea04cdd606982a65

### ソースコードリーディングを通してわかったこと

* express 公式サイト => https://expressjs.com/
* src/views　配下にHTMLを配置している。テンプレートエンジンとしては.pugを採用しているみたい
    * pugとは? => https://kumaweb-d.com/web/basics-of-pug/
    * 他にどんなJSのテンプレートエンジンが存在する？ => https://www.granfairs.com/blog/cto/nunjucks1
    * express で使用できるテンプレートエンジン => https://github.com/expressjs/express/wiki#template-engines
    * ここでこのアプリケーションはpugを使用すると明記している => https://github.com/empenguin1186/TypeScript-Node-Starter/blob/master/src/app.ts#L44
    * テンプレートエンジンの中でTSの変数を使用したい場合は、`render(<pugファイルの名前>, <変数定義>)` で使用する変数を定義し、テンプレートエンジン側では `#{変数}` という形式で使用する
* ルーティングは https://github.com/empenguin1186/TypeScript-Node-Starter/blob/master/src/app.ts で行なっている]
* `res.render(te, {variable})` メソッドはテンプレートエンジンを用いてHTMLを描画するメソッドで、`res.redirect(path)`メソッドは指定されたパスに対応する処理(app.tsで定義されている)を行う。
* src/controller/contact.ts mailOptionsの型定義はどこに?
    * nodemailerライブラリが読み込んでいるMailライブラリの`.d.ts`ファイルに定義されている(node_modules/@types/nodemailer/lib/mailer/index.d.ts)
    * nodemailer公式: https://nodemailer.com/about/
* `default export` について
    * 参考文献: https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/export
    * 外部モジュールを読み込むときに単一の重要なものを読み込む時に使用する。
    以下公式リファレンスより抜粋
    
    > エクスポート方法は、名前付きとデフォルトの 2 種類あります。名前付きエクスポートはモジュールごとに複数持てますが、デフォルトエクスポートは 1 つに限ります。それぞれのエクスポート方法は、上記の構文のひとつに対応します:

    > 名前付きエクスポートは、さまざまな値をエクスポートするのに役立ちます。インポートするときに、対応するオブジェクトと同じ名前を使用しなければなりません。
    > 一方、デフォルトエクスポートは以下のように任意の名前を使用できます:

    例) クラスモジュール(user_commonjs.ts)
    ```TypeScript
    export default class {
    constructor(private _name: string, private _age: number) {

        }
        get name(): string {
            return this._name;
        }
        get age(): number {
            return this._age;
        }
        set name(name: string) {
            this._name = name;
        }
        set age(age: number) {
            this._age = age;
        }

    }
    ```
    default exportではクラス名を持たせないこともできる

    使用する側
    ```TypeScript
    import Hoge from './user_commonjs';

    var user = new Hoge("empenguin1186", 26);
    console.log(user.age);
    ```

    【結論】
    * 名前付きexport はモジュールごとに複数持つことが可能であるが、呼び出し側で使用する時にはモジュールファイルで定義されたメンバの名前を指定する必要がある
    * default export はモジュールごとに1つしか持つことはできないが、呼び出し側で使用する時は指定する名前は自由に決めて良い

### VS Code のショートカットについて
* cmd + P でエディタで開いている別のファイルにジャンプする
* F12 で定義元にジャンプ
* Ctl + 「-」で前編集した箇所に移動 
* key-bind 公式：https://code.visualstudio.com/docs/getstarted/keybindings
* cmd + k, sでインストールされているVS Codeのショートカットが表示される
    * 検索はkey-bind公式に記載されている名前を使用して行うことができる


### python関連メモ
https://qiita.com/yoyoyo_/items/56c6fcbd5a853460f506
https://github.com/yoyoyo-yo/DeepLearningMugenKnock
https://qiita.com/tani_AI_Academy/items/3edc5effeb386ae3caa9

### MDN
https://developer.mozilla.org/ja/




