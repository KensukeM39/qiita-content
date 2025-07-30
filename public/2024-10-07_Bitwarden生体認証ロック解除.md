---
title: Bitwardenの自動入力における生体認証ロック解除の無効化を解決する
tags:
  - Security
  - password
  - bitwarden
private: false
updated_at: '2024-10-07T00:28:12+09:00'
id: 422ccb04ae9a07dfe16e
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

iPhone で Bitwarden を使っているときに、以下のような問題が発生しました。
- ID とパスワードの自動入力に Bitwarden を選択したら、**エラーメッセージ**と共に**マスターパスワードの入力**を要求された🚨
- マスターパスワードの入力後、**ポップアップが表示され**、「続行」を選択したが**自動入力できなかった**

この問題の原因を突き止めて解決することが出来たので、その過程と解決方法についてまとめてみました。

　
セキュリティ技術等に詳しくないという方でも理解できるように、わかりやすく表現しているので、厳密性に欠いている部分もあるかもしれません。

もし誤りや解決方法にリスクがある場合は、編集リクエストを送ってくださると、勉強にもなりとても助かります！


## 結論

問題：Bitwarden で自動入力をするときに、生体認証ロック解除がうまく機能しなかった
原因：KDF メモリを大きな値に設定したことによる、メモリ不足
解決方法：ウェブアプリ版 Bitwarden で、KDF メモリを含む暗号化の設定を変更する

## 解決するまでの過程

問題が発生した環境はこんな感じです。
- 機種：iPhone 12 mini
- バージョン：iOS 17.7
- Bitwarden のバージョン：2024.9.2

いつもなら、Bitwarden の自動入力を選択したら、生体認証ロック解除によりスムーズに自動入力できていました。

しかし、なぜか生体認証ロックが無効化されていたため、マスターパスワードの入力を要求されました。

> このアカウントの自動入力の生体認証ロック解除はマスターパスワードの検証待ちで無効になっています。

|　　　　　　　　  　　　　　　　　   　　　　　　　　　　　　　　　　　　　　　|
|:-:|
|<img width="250" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3902633/c38b138e-c4dd-e1cd-d31a-7397fee86650.jpeg">|
<font color="gray">見覚えのないエラーメッセージ</font>

[自動入力の生体認証ロック解除の有効化する方法](https://bitwarden.com/ja-jp/help/biometrics/#:~:text=%E3%83%9E%E3%82%B9%E3%82%BF%E3%83%BC%E3%83%91%E3%82%B9%E3%83%AF%E3%83%BC%E3%83%89%E3%81%AE%E7%A2%BA%E8%AA%8D%E5%BE%85%E3%81%A1%E3%81%A7%E7%84%A1%E5%8A%B9%E5%8C%96%E3%81%95%E3%82%8C%E3%81%BE%E3%81%97%E3%81%9F)が Bitwarden のヘルプに記載されていたので、手順通りにしたものの、解決しませんでした。



## メモリ不足問題

なにか原因を発見するための手がかりがないか確認していると、ポップアップに「メモリ不足により」と書かれていました。

|　　　　　　　　  　　　　　　　　   　　　　　　　　　　　　　　　　　　　　　|
|:-:|
|<img width="247" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3902633/a42740ac-6906-fb6b-f203-c523ea821855.jpeg">|
<font color="gray">マスターパスワード入力後に表示されたポップアップ</font>

メモリの使用量を設定できないか調べてみましたが、iOS 版と Chrome 拡張機能版にはありませんでした。

唯一、ウェブアプリ版の Bitwarden であれば変更できることがわかりました！

　
以下のリンクから、ウェブアプリ版 Bitwarden に移動できます。

https://vault.bitwarden.com/#/login

ログインしたら、左側のナビゲーションメニューから「設定」→「セキュリティ」を選択し、上部のタブメニューにある「キー」を選択してください。

今回変更した設定は「KDF アルゴリズム」「KDF メモリ」「KDF 反復回数」「KDF 並列性」の4つです。

## KDF ってなに？

セキュリティ技術に疎いので KDF というものを初めて知ったんですけど、色々と調べた結果、僕なりに解釈した上でわかりやすく説明していきます。

まず、Bitwarden ではユーザーアカウントを認証するシステム内に KDFs を2回使用しています。

1回目は、僕らが設定したマスターパスワードと一緒にメールアドレスを KDFs に入れることで、マスターキーを生成しています。

2回目は、マスターキーと一緒にマスターパスワードを KDFs に入れることで、マスターパスワードハッシュを生成しています。
　
![bitwarden-encryption-process.png](https://res.cloudinary.com/bw-com/image/upload/f_auto/v1/ctf/7rncvj1f8mw7/1rLMJoZFka4Per5lIyuMv9/33bc3f62358591bfe4cb86d3c3375535/whitepaper-acctcreate.png?_a=DAJAUVWIZAAB, "パスワードのハッシュ化、キー導出、および暗号化プロセスの概要")
<font color="gray">引用：[Bitwarden セキュリティホワイトペーパ](https://bitwarden.com/ja-jp/help/bitwarden-security-white-paper/#:~:text=%E4%BB%A5%E4%B8%8B%E3%81%AB%E3%80%81%E3%83%91%E3%82%B9%E3%83%AF%E3%83%BC%E3%83%89%E3%81%AE%E3%83%8F%E3%83%83%E3%82%B7%E3%83%A5%E5%8C%96%E3%80%81%E3%82%AD%E3%83%BC%E5%B0%8E%E5%87%BA%E3%80%81%E3%81%8A%E3%82%88%E3%81%B3%E6%9A%97%E5%8F%B7%E5%8C%96%E3%83%97%E3%83%AD%E3%82%BB%E3%82%B9%E3%81%AE%E6%A6%82%E8%A6%81%E3%82%92%E7%A4%BA%E3%81%97%E3%81%BE%E3%81%99%E3%80%82)</font>

　
KDFs とは鍵導出関数（Key Derivation Functions）と呼ばれるもので、可変長のパスワードをある計算方法に従って固定長のキーに変換する関数のことです。

わかりやすい表現にすると、長さ（文字数）が様々なパスワードを特定の計算方法に従って、一定の長さの値に変換する変換器ということです。

イメージとしては、変形自在であるたい焼きの生地を、たい焼き器に流し込むことで、決まった形のたい焼きするという感じですかね。

　
そして、Bitwarden では2種類の KDF アルゴリズムを選択できます。1つ目が PBKDF2、2つ目が Argon2id。

## PBKDF2

- [NIST（米国国立標準技術研究所）に推奨されている](https://pages.nist.gov/800-63-3/sp800-63b.html#memsecretver)
- [FIPS 140-2](https://www.tjsys.co.jp/embedded/encryption/about-fips140.htm) 準拠が必要な場合はPBKDF2
- KDF 反復回数とは、KDF が内部で行う処理を繰り返す回数のこと
- KDF 反復回数を増やすと、処理時間が長くなる
    ⇒　攻撃者によるパスワードクラッキング攻撃に時間がかかる
    ⇒　安全性が高まる
    ⇒　ログインするのに必要な時間も長くなってしまう
    
　
※FIPS 140-2：NIST が策定した暗号モジュール（ハードウェアやソフトウェアを含む）のセキュリティ要件に関する米国連邦標準規格

※パスワードクラッキング攻撃：不正なアクセスを目的として、ユーザーのパスワードを解読しようとする攻撃手法

## Argon2id

- Argon2アルゴリズムは、2015年[パスワードハッシュ化コンペティション](https://github.com/p-h-c/phc-winner-argon2)の優勝者
- Argon2id は、Argon2アルゴリズムのうちの1つ（Argon2i, Argon2d, Argon2id）
- Argon2id は、Argon2i と Argon2d のハイブリッド
- サイドチャンネルキャッシュタイミング攻撃と GPU クラッキング攻撃への耐性が高い

　
※サイドチャンネルキャッシュタイミング攻撃：暗号システムの実装における脆弱性を悪用する高度な攻撃手法

※GPU クラッキング攻撃：GPU の並列処理能力を利用してパスワードやハッシュ値を高速に解読しようとする攻撃手法

## どっちを選べばいいの？

[Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) にこのような記載があります。

> While **Argon2id should be the best choice for password hashing**, scrypt should be used when the former is not available.
（**パスワードハッシュには Argon2id が最適**ですが、それが利用できない場合は scrypt を使用すべきです）

> Since PBKDF2 is recommended by NIST and has FIPS-140 validated implementations, so it should be the preferred algorithm **when these are required**.
（PBKDF2はNISTにより推奨されており、FIPS-140認証済みの実装もあるため、**これらが必要な場合は**、このアルゴリズムが推奨されるべきである。）

要するに、パスワードハッシュとして **Argon2id が最適**。
ただし、**FIPS-140準拠が必要な場合は PBKDF2** を推奨する。

# 設定を変更しよう！

ここからは、それぞれの推奨設定についてのお話です。
先ほど誘導したウェブアプリ版 Bitwarden の「**暗号化の設定**」を見てください。

## 注意事項

「KDF の変更」をする前に注意点として、
- データの損失防止のために、必ず保管庫をエクスポートしておく（ナビゲーションの「ツール」から）
- 全てのデバイスでログアウトされるので、マスターパスワードと2段階認証の準備をしておく

## Argon2id の場合

- KDF メモリ：実行時に使用するメモリ量
- KDF 反復回数：同じ処理を繰り返す回数
- KDF 並列性：並列で実行する CPU のスレッド数（同時に動かす作業の数）

　
OWASP は、これらの構成設定を次のように推奨しています。

| KDF メモリ(MiB) | KDF 反復回数 | KDF 並列性 |
| ---: | ---: | ---: |
| 46 | 1 | 1 |
| 19 | 2 | 1 |
| 12 | 3 | 1 |
| 9 | 4 | 1 |
| 7 | 5 | 1 |

これらの構成設定は同等の防御レベルであり、RAM（メモリ）と CPU にはトレードオフの関係があります。

　
Bitwarden のデフォルト設定は、OWASP の推奨設定よりも高くしています。

|  | KDF メモリ(MiB) | KDF 反復回数 | KDF 並列性 |
| --- | ---:| ---: | ---: |
| Bitwarden のデフォルト設定 | 64 | 3 | 4 |
| 僕の場合 | 100 | 10 | 8 |

　
僕はなぜかこれらの数値を高く設定していたので、生体認証ロック解除がうまく機能せず、「メモリ不足が原因」というポップアップが出たということですね...！（謎）

ちなみに、[Bitwarden 公式のヘルプ](https://bitwarden.com/ja-jp/help/kdf-algorithms/#:~:text=iOS%E3%81%AF%E3%82%AA%E3%83%BC%E3%83%88%E3%83%95%E3%82%A3%E3%83%AB%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AE%E3%82%A2%E3%83%97%E3%83%AA%E3%83%A1%E3%83%A2%E3%83%AA%E3%82%92%E5%88%B6%E9%99%90%E3%81%97%E3%81%BE%E3%81%99%E3%80%82%E3%83%87%E3%83%95%E3%82%A9%E3%83%AB%E3%83%88%E3%81%AE64%20MB%E3%81%8B%E3%82%89%E3%82%A4%E3%83%86%E3%83%AC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E5%A2%97%E3%82%84%E3%81%99%E3%81%A8%E3%80%81%E8%87%AA%E5%8B%95%E5%85%A5%E5%8A%9B%E3%81%A7%E4%BF%9D%E7%AE%A1%E5%BA%AB%E3%82%92%E3%83%AD%E3%83%83%E3%82%AF%E8%A7%A3%E9%99%A4%E3%81%99%E3%82%8B%E9%9A%9B%E3%81%AB%E3%82%A8%E3%83%A9%E3%83%BC%E3%81%8C%E7%99%BA%E7%94%9F%E3%81%99%E3%82%8B%E5%8F%AF%E8%83%BD%E6%80%A7%E3%81%8C%E3%81%82%E3%82%8A%E3%81%BE%E3%81%99%E3%80%82)に今回の問題についてしっかりと書いてありました笑

> iOS はオートフィルのためのアプリメモリを制限します。デフォルトの64 MB からイテレーションを増やすと、自動入力で保管庫をロック解除する際にエラーが発生する可能性があります。

　
実際に数値を設定する際は、Bitwarden のデフォルト設定に従いましょう。

それでも「メモリ不足」とポップアップが出る場合は、KDF メモリを減らして、KDF 反復回数を増やして調整していきましょう（KDF 反復回数を1増やすのに対して、どれくらい KDF メモリを減らせばいいかは書かれていませんでした）。

　
「KDF 並列性」に関しては、Bitwarden を使用するコンピュータ（デバイス）の CPU のスレッド数によって上限が決まります。

Bitwarden のデフォルト設定に従えば問題ないと思います。


## CPU のスレッド数を調べ方（iPhone, iPad の場合）

1. デバイスの機種名を確認
    
    iPad の「設定アプリ」→「一般」→「情報」→「機種名：iPad 8th Gen」
　
2. 機種ごとに Apple 公式の技術仕様を調べて、内臓チップの名称を確認
    
    iPad 8th Gen の場合：「iPad 8 技術仕様」で検索　→　 Apple の「iPad（第8世代）-技術仕様」を確認　→　内臓チップ「A12 Bionic」と判明
　  
3.「[内臓チップの名称]　スレッド数」で検索。検索結果から適当にサイトを開き、「スレッド数（Threads）」が書かれていないか確認

    A12 Bionic の場合：「CPU コア / Threads：6/6」→「スレッド数は6」
    （[cpu-monkey より](https://www.cpu-monkey.com/ja/cpu-apple_a12_bionic "Apple A12 Bionic ベンチマーク、テスト、および仕様")）
    

## PBKDF2の場合

Bitwarden では、以下の内容に従うよう勧められています。

- KDF 反復回数は600,000回以上（OWASP による推奨）
- 値を100,000単位で増やし、Bitwarden を使用するすべてのデバイスでテストする

## 自動入力の有効化

Bitwarden の KDF 設定を変更し、ログインが終わったら、自動入力を有効化していきましょう！

　
1. Bitwarden 上で、「設定」→「アカウントのセキュリティ」→「ロック解除オプション」→「Face ID でロック解除：オフ」にする。
　
2. iOS の設定アプリを開いて、「パスワード」→「パスワードオプション」→「Bitwarden：オフ」→「パスワードとパスキーを自動入力：オフ」にする。
　
3. Bitwarden 上に戻り、「Face ID でロック解除：オン」にする。
　
4. iOS の設定アプリに戻り、「パスワードとパスキーを自動入力：オン」→「Bitwarden：オン」にする。
　
5. Bitwarden のマスターパスワードを要求されるので入力する。

|　　　　　　　　  　　　　　　　　   　　　　　　　　　　　　　　　　　　　　　|
|:-:|
|<img width="247" src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3902633/43ee7b23-0ed4-b6c8-b8e6-0fe955b2cd5b.jpeg">|
<font color="gray">自動入力が有効化された場合</font>

画像のような画面に切り替わり、実際に自動入力ができたら完了です。
おつかれさまでした！

## 最後に

学習したり経験したことを言語化する・アウトプットすることは、知恵を身につけるための手段としてもとても良いものであり、今回の問題について Google 先生で調べてもなかなか出てこなかったので、試しに Qiita にまとめてみました！

　
僕は今まで暗号化とかハッシュ化とか難しそうで毛嫌いしていて、セキュリティ技術に疎い人間の一人です🫠笑

今回の問題解決とその内容を Qiita にまとめるにあたって詳しく調べていくうちに、理解するのに時間がかかりましたが、楽しみながらセキュリティへの理解を深めることができて、とてもいい経験になりました！

　
Bitwarden の仕組みや KDF（鍵導出関数）について詳しく知りたい方は、以下のページも見てみてください👍

https://tex2e.github.io/blog/crypto/cryptography-hash-functions

https://tex2e.github.io/blog/crypto/mac-and-key-derivation

## 参考
- Bitwarden の仕組み
[Bitwardenセキュリティホワイトペーパ | Bitwarden ヘルプセンター](https://bitwarden.com/ja-jp/help/bitwarden-security-white-paper/)
- Bitwarden での生体認証
[生体認証でロック解除 | Bitwarden ヘルプセンター](https://bitwarden.com/ja-jp/help/biometrics/)
- Bitwarden の暗号化の設定
[KDFアルゴリズム | Bitwarden ヘルプセンター](https://bitwarden.com/ja-jp/help/kdf-algorithms/)
- パスワードハッシュアルゴリズム（KDFアルゴリズム）
[Password Storage - OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- 「Password Storage - OWASP Cheat Sheet Series」の日本語訳
[OWASPのチートシートを読んでパスワードの保存方法を確認する - Zenn](https://zenn.dev/maronn/articles/read-hash-in-owasp)
- Argon2
[P-H-C/phc-winner-argon2: The password hash Argon2, winner of PHC - GitHub](https://github.com/p-h-c/phc-winner-argon2)
