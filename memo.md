# 概要
この資料は README.md を読んでまとめた資料である

## TypeScript + Node
### Project Structure

ファイル構成は以下の通り
```
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

```:JSON
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

```:JSON
"devDependencies": {
    "@types/async": "^3.0.0",
    "@types/bcrypt-nodejs": "^0.0.30",
    "@types/bluebird": "^3.5.27",
    ...
```

`npm install`を実行すると型定義ファイルは `./node_modules/@types` 配下に配置される

例
```
ls -l node_modules/@types/async/
total 56
-rw-r--r--  1 kodaira  YAHOO\Domain Users   1183  6  5 05:54 LICENSE
-rw-r--r--  1 kodaira  YAHOO\Domain Users    757  6  5 05:54 README.md
-rw-r--r--  1 kodaira  YAHOO\Domain Users  15765  6  5 05:54 index.d.ts // これが型定義ファイル
-rw-r--r--  1 kodaira  YAHOO\Domain Users   2148  7  8 08:27 package.json
```

### What if a library isn't on DefinitelyTyped?




