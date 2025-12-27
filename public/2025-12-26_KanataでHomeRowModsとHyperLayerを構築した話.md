---
title: Rust製ツール「Kanata」× Home Row Mods × Hyper Layer により操作性・作業効率の向上を目指すカスタムキーマップ構築録
tags:
  - Rust
  - macOS
  - Keyboard
  - kanata
  - HomeRowMods
private: false
updated_at: '2025-12-26T04:13:36+09:00'
id: 2030532d6cd4ec248281
organization_url_name: null
slide: false
ignorePublish: false
---
## はじめに

エンジニアや研究者にとって、キーボードは身体の一部と言っても過言ではありません。しかし、既製品のキーボード配列には、長年の「矛盾」が存在します。

* **頻繁に使うキー（Backspace, Esc, 矢印キー）が、ホームポジションから遠い**
* **一等地（Aの隣）にある CapsLock が、ほとんど使われていない**
* **JIS配列とUS配列を行き来すると、指が混乱する**

「ハードウェアの制約に自分を合わせるのではなく、ソフトウェアで自分に最適化したい」。
そう考えた私は、macOS の定番である Karabiner-Elements ではなく、**Rust製のOSSキーマッピングツール「[Kanata](https://github.com/jtroo/kanata)」** を選択しました。

本記事では、JIS/US 配列が混在する環境において、**Home Row Mods**（ホームポジション修飾キー）やレイヤー機能を駆使し、「どのキーボードでも同じ操作感」を実現するための設計思想と実装の記録を共有します。

## 1. なぜ Karabiner ではなく Kanata なのか？

macOS でキーボードカスタマイズといえば **Karabiner-Elements** がデファクトスタンダードです。GUI で設定でき、非常に優秀なツールですが、私の環境ではいくつかの課題がありました。

1.  **クロスプラットフォーム対応:**
    Karabiner は macOS 専用です。将来的に他のOSでも同じ操作体系（設定ファイル）を流用したいと考えた際、OS 依存はネックになります。
2.  **設定ファイルによる管理:**
    「Shiftキーを2回タップして長押ししたら Function レイヤー」といったような、文脈に依存した複雑な挙動を定義しようとした際、Kanata の方が設定ファイル（`.kbd`）で宣言的に記述しやすいと感じました。Git管理との相性も良好です。

そこで出会ったのが **Kanata** です。設定ファイルがコードベースで管理でき、挙動を細かく制御できる点に惹かれました。

## 2. 設計思想：ホームポジションから「一歩」も動かない

今回のキーマップ構築における最大のテーマは、「**思考と指の動きを一致させること**」です。物理的なキー配列が JIS でも US でも、指の動きは統一します。

### 2-1. レイヤー構成のイメージ

論より証拠、実際に構築したレイヤーの配置をご覧ください。US配列でのイメージですが、JISでも操作感は統一しています。

**Base Layer (通常時)**
Home Row Mods を採用し、ホームポジションに修飾キーを埋め込んでいます。
![Base Layer](https://github.com/user-attachments/assets/72b7947c-bde7-4dbf-a4ee-bf32fdd8c08f)

**Hyper Layer (CapsLock 長押し)**
矢印キーや削除系キーを左手側に配置し、ホームポジションから手を動かさずにカーソル操作を完結させます。
![Hyper Layer](https://github.com/user-attachments/assets/74481466-a1c9-499d-9eb1-6c4607aefbbb)

**Function Layer (左 Shift 2回タップ+長押し)**
ファンクションキーやテンキー操作を集約しています。
![Function Layer](https://github.com/user-attachments/assets/ce930bb3-f350-436b-a04c-9392cf7fb570)

### 2-2. Home Row Mods とは？

修飾キー（Cmd, Ctrl, Shift, Alt）を打つために、小指を不自然に曲げたり、大きく手を動かしていませんか？
**Home Row Mods** は、ホームポジション（A, S, D, F, J, K, L, ;）に修飾キーの役割を持たせる手法です。

* **Tap（短押し）:** 文字入力（A, S...）
* **Hold（長押し）:** 修飾キー（Cmd, Shift...）

これにより、「**文字を打つ指**」のまま「**ショートカットも打てる**」ようになり、小指の負担が軽減されます。

## 3. 実装：配列の壁を超える `.kbd` 設計

Kanata の設定ファイル（`.kbd`）は1つにまとめることも可能ですが、私はあえて**物理配列ごとに設定ファイルを分離**しました。

* `kanata-mac-mac-JIS.kbd` （Apple JIS配列用）
* `kanata-mac-mac-US.kbd` （Apple US配列用）
* `kanata-mac-win-JIS.kbd` （Windows用JISキーボードをMacに繋ぐ用）

### こだわりの設定ポイント

実際の `.kbd` ファイルから、特にこだわった部分を抜粋して解説します。

#### ① Home Row Mods の定義 (defalias)

誤入力を防ぐため、`tap-hold-release-keys` を使用しています。これは「他のキーが押されたタイミング」なども考慮して Tap/Hold を判定してくれる高度な設定です。

```lisp:kanata-mac-mac-US.kbd
(defvar
    ;; 数値(時間)の単位: ms
    tap-time    200
    hold-time   175
    
    left-hand-keys ( q w e r t a s d f g z x c v b )
    right-hand-keys ( y u i o p h j k l ; n m , . / )
)

(defalias
    ;; 左手：Aキーを長押しでCtrl、短押しでa
    a (tap-hold-release-keys $tap-time $hold-time (multi a @tap) lctl $left-hand-keys)
    s (tap-hold-release-keys $tap-time $hold-time (multi s @tap) lalt $left-hand-keys)
    d (tap-hold-release-keys $tap-time $hold-time (multi d @tap) lsft $left-hand-keys)
    f (tap-hold-release-keys $tap-time $hold-time (multi f @tap) lmet $left-hand-keys)
    
    ;; 右手も同様に対称配置
    j (tap-hold-release-keys $tap-time $hold-time (multi j @tap) rmet $right-hand-keys)
    k (tap-hold-release-keys $tap-time $hold-time (multi k @tap) rsft $right-hand-keys)
    l (tap-hold-release-keys $tap-time $hold-time (multi l @tap) ralt $right-hand-keys)
)

```


:::note info
**調整の柔軟性**
`defvar` で定義した `tap-time` などの変数は、キーボードの物理的な特性（キーストロークの深さやスイッチの重さ）に合わせて自由に調整できます。現在はJIS/USともに200msで運用していますが、環境や好みに応じてミリ秒単位で打鍵感をチューニングできるのはKanataの大きな魅力です。
:::

#### ② Hyper レイヤーの活用（入力切替とナビゲーション）

Hyper レイヤー（Capslock長押し）には、カーソル移動だけでなく「入力ソースの切り替え」という重要な役割も持たせています。

**日本語 / 英語入力の切り替え**
US配列などでは「入力切替」のキー確保に悩みますが、右 Cmd キー（`rmet`）を活用して解決しました。

* **Base レイヤー（通常時）:** 英語入力（`Lang2` / 英数）
* **Hyper レイヤー（長押し時）:** 日本語入力（`Lang1` / かな）

これにより、同じキーでも「そのまま押せば英語」「Hyperキーと一緒に押せば日本語」という直感的な切り替えが可能になり、今どちらの入力なのか確認する手間がなくなります。

```lisp:kanata-mac-mac-US.kbd
(defalias
    ...
    en Lang2
    ja Lang1
    ...
)

;; --- Baseレイヤー ---
(deflayer base
    ...
    _       _       _       _               _                       @en     fn
)

;; --- Hyperレイヤー ---
(deflayer hyper
    ...
    _       _       _       _               ret                     @ja     fn
)

;; --- Functionレイヤー ---
(deflayer function
    ...
    _       _       _       _               _                       @ja     fn
)

```

**範囲選択の最適化**
プログラミングや文書作成において頻発する「Shift + ナビゲーションキー」による範囲選択も、ホームポジションを崩さずに行えます。

* **右手:** 中指で `K` をホールド（Shift）
* **左手:** 小指で `Capslock` をホールド（Hyper） + `S/D/F`（矢印）
  ※ Home Row Mods → Hyper Layer という手順でないと上手く機能しない可能性があります。

```lisp:kanata-mac-mac-US.kbd
(deflayer hyper
    _       _       _       _       _       _       _       _       _       _       _       _       _
            _       _       _       _       _       _       _       _       _       _       _       _       _
    esc     C-tab   bspc    up      del     C-S-tab _       _       _       _       _       _       _       _
    @hyper  home    lft     down    rght    end     _       @j      @k      @l      @;      _       _
    @funct  pgup    M-z     M-S-z   pgdn    _       _       _       _       _       _       _
    _       _       _       _               ret                     @ja     fn
)
```

修飾キー（Shift）とナビゲーションキー（矢印キー, Home, End など）の役割を左右の手で分担することで、指を不自然に捻ることなく、流れるようにコードの範囲選択が可能です。


#### ③ 特殊なレイヤー呼び出し (tap-dance)

Function レイヤーは常時使うものではないため、誤爆を防ぐために少し特殊な呼び出し方を採用しました。「左 Shift を2回トントンと叩いて、そのまま押しっぱなしにした時だけ」レイヤーが有効になります。

```lisp
(defalias
    ;; タップor長押し→Left shift
    ;; 2回タップ+長押し→Functionレイヤー
    funct (tap-dance $double-tap-time (lsft (layer-while-held function)))
)

```

この `tap-dance` 機能のおかげで、普段の Shift キー入力（大文字入力など）を邪魔することなく、特殊機能を追加できています。

#### ④ 物理配列の違いを吸収 (defsrc)

JISとUSでは記号の位置が異なりますが、`defsrc`（物理キーの定義）側でその差分を吸収し、レイヤー定義（論理配列）はできるだけ思想を統一しています。

**JIS配列の場合:**

```lisp:kanata-mac-mac-JIS.kbd
(defsrc
    ...
    tab     q       w       e       r       t       y       u       i       o       p       ⎡       ⎣
    lctl    a       s       d       f       g       h       j       k       l       ;       apo     \       ret
    ...
)

```

**US配列の場合:**

```lisp:kanata-mac-mac-US.kbd
(defsrc
    ...
    tab     q       w       e       r       t       y       u       i       o       p       ⎡       ⎣       \
    caps    a       s       d       f       g       h       j       k       l       ;       apo     ret
    ...
)

```

## 4. 導入方法とトラブルシューティング

### インストール

macOS では **Homebrew** を利用してインストールしました。

```bash
brew install kanata
```

※ 権限周りの設定やドライバ（Karabiner-VirtualHIDDevice）の導入が必要です。詳細は公式ドキュメントまたは私のリポジトリの README を参照してください。

### ハマりどころ：スタートアップ時の挙動

Macbook Air の方では問題ないのですが、Mac Miniで導入した際に起動直後のログイン画面において、「**約5秒経過すると『i』などの定義されているキーが反応しなくなり、その後ログインしてもKanataは起動していない状態になる**」 という現象に遭遇しました。原因は不明ですが、機種の問題というより、導入手順や設定に問題があるとみています。

現状の回避策としては、「**ログイン画面が表示されたら即座にパスワードを入力する（Kanataが干渉する前にログインを完了させる）**」ことで対応しています。根本的な解決には至っていませんが、運用でのカバーは可能です。

### ハマりどころ：Home Row Mods の「誤爆」

「普通に文字を打ったつもりが、Cmdキーとして認識された」
これは Home Row Mods 導入時の最大の壁です。前述の `defvar` で定義した `tap-time` や `hold-time` を自分のタイピング速度に合わせて 10ms 単位で調整しました。最初は長めの設定（200ms〜）から始め、慣れに合わせて短くしていくのがおすすめです。

## 5. 結論

### 慣れるまでの「1週間」を乗り越える

正直に言うと、導入直後は違和感しかありませんでした。特に Home Row Mods は、タイピングのリズムを変える必要があるため、無意識に使えるようになるまで約1週間かかりました。

しかし、一度慣れてしまうと世界が変わります。

* **カーソル移動がスムーズ:** ホームポジションから手を動かさずに、矢印キーや PageUp/Down が打てる。
* **手首の疲れが軽減:** 手首を左右に振る動作が減った。
* **配列を選ばない:** 設定ファイルを切り替えるだけで、どんなキーボードでも「いつもの操作感」になる。

今回の構築を通じて、キーボード配列は「与えられるもの」ではなく「**自分の使い方に合わせて育てていくもの**」だと実感しました。
本記事の設定は macOS 向けですが、Kanata 自体は Windows にも対応しています（※私はWindows版を作成する予定はありませんが、興味のある方はぜひ挑戦してみてください！）。

もし今のキーボード操作に少しでも違和感があるなら、ぜひ Kanata で「自分だけの快適なキーマップ」を作り上げてみてください。


## 8. 参考資料・リンク集

* [今回開発したリポジトリ (GitHub)](https://github.com/KensukeM39/kanata-homerow-hyper-keymap)
  * 実際の `.kbd` ファイルやインストール手順はこちらで公開しています。

* [Kanata (GitHub)](https://github.com/jtroo/kanata)
* [Kanata 設定ドキュメント (config.adoc)](https://github.com/jtroo/kanata/blob/main/docs/config.adoc)
