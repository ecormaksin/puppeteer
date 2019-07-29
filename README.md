https://github.com/GoogleChrome/puppeteer
https://qiita.com/hashrock/items/15f4a4961183cfbb2658
https://techacademy.jp/magazine/16151
https://github.com/GoogleChrome/puppeteer/issues/3451
https://github.com/GoogleChrome/puppeteer/blob/master/docs/troubleshooting.md#setting-up-chrome-linux-sandbox

# Puppeteerとは？

- Nodeのライブラリ
- ディベロッパー ツールのプロトコル経由でChromeまたはChromiumを制御する高レベルのAPIを提供する。
- デフォルトではヘッドレス（GUIを起動することなくコマンドライン）で実行するが、GUIで実行するよう設定することもできる。
- ブラウザで手動で行えるほとんどのことをPuppetterから行える。
    - 例
        - ページのスクリーンショットやPDFを生成する。
        - SPA（シングル ページ アプリケーション）をクローリング（データ収集）して、事前にレンダリングされたコンテンツを生成する（つまりサーバーサイドのレンダリング）。
        - フォームのサブミット、UIテスト、キー入力などを自動化する。
        - 最新の自動化テスト環境を生成する。最新のJavaScriptとブラウザの機能を使って、最新版のChromeで直接テストを実行できる。
        - パフォーマンス問題の診断に役立つサイトの時系列での追跡を可能にする。
        - Chromeの拡張機能をテストする。

# Puppeteerのインストール

## Nodeプロジェクトのセットアップ

- PuppeteerはNodeのライブラリなので、インストールの前にNodeプロジェクトのセットアップが必要。

```
mkdir ~/puppeteer_test # 任意のプロジェクトディレクトリ
cd ~/puppeteer_test

npm init
```

- すべてのプロンプトで未入力（デフォルト）のままEnter

```
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (puppeteer_test)
version: (1.0.0)
description:
entry point: (index.js)
test command:
git repository:
keywords:
author:
license: (ISC)
About to write to /home/vagrant/puppeteer_test/package.json:

{
  "name": "puppeteer_test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}


Is this OK? (yes)
```

```
npm i puppeteer
```

- Puppeteerをインストールすると、最新版のChromiumがダウンロードされる。このChromiumはPuppeteerのAPIバージョンに対応しており、APIが動作することが保証されている。ダウンロードをスキップする場合は[環境変数](https://github.com/GoogleChrome/puppeteer/blob/v1.18.0/docs/api.md#environment-variables) を参照。

- 以下のような出力になった。

```

> puppeteer@1.18.0 install /home/vagrant/puppeteer_test/node_modules/puppeteer
> node install.js

Downloading Chromium r669486 - 347.1 Mb [====================] 100% 0.0s
Chromium downloaded to /home/vagrant/puppeteer_test/node_modules/puppeteer/.local-chromium/linux-669486
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN puppeteer_test@1.0.0 No description
npm WARN puppeteer_test@1.0.0 No repository field.

+ puppeteer@1.18.0
added 43 packages from 22 contributors and audited 50 packages in 39.722s
found 0 vulnerabilities
```

## puppeteer-core

- バージョン1.7から`puppeteer-core`パッケージがリリースされた。デフォルトではChromiumをダウンロードしない。
- Puppeteerの軽量版
- インストール済みのブラウザを起動したり、リモートのブラウザに接続したりする。
- 利用するブラウザとpuppeteer-coreのバージョンに互換性が必要。
- 詳細は[puppeteer vs puppeteer-core](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteer-vs-puppeteer-core) を参照。

# 使い方

- PuppeteerはNode6.4.0以上が必要。※下記サンプルは7.6.0以降からサポートしている`async/await`を使っている。
- 使い方の概要
    1. ブラウザのインスタンスを生成する。
    2. ページを開く。
    3. APIで操作する。

## サンプルの実行

-  **サンプル通りにjsファイルを作って実行するだけだと `UnhandledPromiseRejectionWarning: Error: Failed to launch chrome! 〜 No usable sandbox! 〜 ` のエラーが発生したので、上記参考URLを元にsetuid sandboxをセットアップする。（「[recommended] Enable user namespace cloning」はファイルが見つからずエラーになった。）**

```
cd ./node_modules/puppeteer/.local-chromium/linux-669486/chrome-linux/

sudo chown root:root chrome_sandbox
sudo chmod 4755 chrome_sandbox

sudo cp -p chrome_sandbox /usr/local/sbin/chrome-devel-sandbox

echo "export CHROME_DEVEL_SANDBOX=/usr/local/sbin/chrome-devel-sandbox" >> ~/.bashrc
source ~/.bashrc
```

### ~/puppeteer_test/example.js（スクリーンショットの取得）

```
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://example.com');
  await page.screenshot({path: 'example.png'});

  await browser.close();
})();
```

```
cd ~/puppeteer_test
node example.js
```

- `~/puppeteer_test/example.png`が生成されていることを確認する。

- Puppeteerが開くブラウザのページサイズのデフォルトは800px x 600px。ページサイズは[Page.setViewport()](https://github.com/GoogleChrome/puppeteer/blob/v1.18.0/docs/api.md#pagesetviewportviewport) で変更できる。

### ~/puppeteer_test/hn.js（PDFの作成）

- **pdfの生成はヘッドレスでない場合にエラーが発生する。（[参照元](https://github.com/GoogleChrome/puppeteer/issues/830)）**

```
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://news.ycombinator.com', {waitUntil: 'networkidle2'});
  await page.pdf({path: 'hn.pdf', format: 'A4'});

  await browser.close();
})();
```

```
cd ~/puppeteer_test
node hn.js
```

- `~/puppeteer_test/hn.pdf`が生成されていることを確認する。

- PDF生成の詳細は[Page.pdf()](https://github.com/GoogleChrome/puppeteer/blob/v1.18.0/docs/api.md#pagepdfoptions) を参照。

### ~/puppeteer_test/get-dimensions.js（ページの状況を評価するスクリプト）

```
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://example.com');

  // Get the "viewport" of the page, as reported by the page.
  const dimensions = await page.evaluate(() => {
    return {
      width: document.documentElement.clientWidth,
      height: document.documentElement.clientHeight,
      deviceScaleFactor: window.devicePixelRatio
    };
  });

  console.log('Dimensions:', dimensions);

  await browser.close();
})();
```

```
cd ~/puppeteer_test
node get-dimensions.js
```

# デフォルトの実行時設定

## 1. ヘッドレスモード

- GUIありで実行する場合は`{headless: false}`を指定する。（デフォルトは`true`）

```
const browser = await puppeteer.launch({headless: false});
```

## 2. バンドル版のChromiumを使用

- 他のバージョンのChromeやChromiumを使う場合は、ブラウザ インスタンスを生成する時に実行パスを指定する。

```
const browser = await puppeteer.launch({executablePath: '<対象となるChrome／Chromiumのパス>'});
```

- 詳細は[Puppeteer.launch()](https://github.com/GoogleChrome/puppeteer/blob/v1.18.0/docs/api.md#puppeteerlaunchoptions) を参照。

- ChromeとChromiumの違いについては[この記事](https://www.howtogeek.com/202825/what%E2%80%99s-the-difference-between-chromium-and-chrome/) を参照。

- Linuxユーザーに対しては[この記事](https://chromium.googlesource.com/chromium/src/+/master/docs/chromium_browser_vs_google_chrome.md) でいくつかの違いを説明している。

## 3. まっさらなユーザープロファイルの生成

- Puppeteerは独自のChromiumのユーザー プロファイルを生成する。実行の度にクリーンアップされる。

# 資料

- [API ドキュメント](https://github.com/GoogleChrome/puppeteer/blob/v1.18.0/docs/api.md)
- [例](https://github.com/GoogleChrome/puppeteer/tree/master/examples/)
- [資料の一覧](https://github.com/transitive-bullshit/awesome-puppeteer)

# デバッグのTips

1. ヘッドレスモードをオフにする。ブラウザが表示している内容を確認することが役に立つこともある。

```
const browser = await puppeteer.launch({headless: false});
```

2. 速度を下げる。`slowMo`オプションはPuppeteerの操作を指定したミリ秒で遅くする。

```
const browser = await puppeteer.launch({
  headless: false,
  slowMo: 250 // slow down by 250ms
});
```

3. コンソールの出力をキャプチャする。`console`イベントを監視できる。`page.evaluate()`でデバッグするのは簡単。

```
page.on('console', msg => console.log('PAGE LOG:', msg.text()));

await page.evaluate(() => console.log(`url is ${location.href}`));
```

4. アプリケーション コード ブラウザでデバッガーを使う

- 実行のコンテキストが2つある。: テストコードを実行しているnode.js、テスト対象のアプリケーションコードを実行しているブラウザ

- アプリケーション コード ブラウザでコードをデバッグできる。つまり、`evaluate()`の中にコードを埋め込む。
    - Puppeteerを起動する時に`{devtools: true}`を使う。
      ```
      const browser = await puppeteer.launch({devtools: true});
      ```
    - デフォルトのテストタイムアウトを変更する。
        - jest: `jest.setTimeout(100000);`
        - jasmine: `jasmine.DEFAULT_TIMEOUT_INTERVAL = 100000;`
        - mocha: `this.timeout(100000);`（`function`を使い、`=>`を使わないようテストを変更する。([stack overflowの該当ページ](https://stackoverflow.com/questions/23492043/change-default-timeout-for-mocha/23492442#23492442) )）
    - `debugger`を使ったevaluateステートメントを追加する／既存のevaluateステートメントに`debugger`を追加する。  
      evaluateステートメントでテスト実行が停止し、Chromiumがデバッグモードで停止する。

5. node.jsでデバッガーを使う

- テストコードをデバッグする。
    - 例
        - node.jsで`await page.click()`をステップオーバーして、アプリケーションコードのブラウザでクリックイベントが起こることが確認できる。

- [Chromiumのバグ](https://bugs.chromium.org/p/chromium/issues/detail?id=833928) のためにディベロッパー ツールで`await page.click()`を実行できない。なので、何かを試したければテストコードのファイルに追加しなければいけない。
    - テストに`debugger;`を追加する。
      ```
      debugger;
      await page.click('a[target=_blank]');
      ```
    - `headless`を`false`にする。
    - `node --inspect-brk`を実行する。例）`node --inspect-brk node_modules/.bin/jest tests`
    - Chromeで`chrome://inspect/#devices`を開き、`inspect`をクリックする。
    - 新しく開かれたテストブラウザで、テスト実行を再開するために`F8`を押す。
    - `debugger`が起動され、テストブラウザでデバッグできる。

6. 詳細なログを有効にする。内部のディベロッパーツールのトラフィックが`puppeteer`の名前空間の元に`debug`をモジュール経由でログが記録される。

```
# 基本的な詳細ロギング
env DEBUG="puppeteer:*" node script.js

# プロトコルのトラフィックには不要なものも含まれている。この例では"Network"のメッセージは除去する。
env DEBUG="puppeteer:*" env DEBUG_COLORS=true node script.js 2>&1 | grep -v "Network"
```

7. [ndb](https://github.com/GoogleChromeLabs/ndb) を使ってPuppeteer(node)のコードを簡単にデバッグする

- `npm install -g ndb`(もっとよいのは[npx](https://github.com/zkat/npx)を使うこと！)
- `debugger`をPuppeteer(node)のコードに追加する。
- テストコマンドの前に`ndb`(または`npx ndb`)を追加する。
    - 例
        - `ndb jest`や`ndb mocha`（または`npx ndb jest` / `npx ndb mocha`）

# FAQの抜粋

**Q: PuppeteerはSelenium/WebDriverを置き換えるか？**
A: 違う。

- Selenium/WebDriverはクロスブラウザの自動化にフォーカスしている。
- PuppeteerはChromiumにフォーカスしている。
