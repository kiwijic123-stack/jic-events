# JICイベントカレンダー 引き継ぎ（CLAUDE.md）

JIC協会のイベントカレンダー公開サイト。運用者＝濱野優輝（ゆうき）さん。
チャットで更新依頼を受けたら、編集→テスト→push まで行う（ユーザー側の操作は不要）。

## 基本情報

| 項目 | 内容 |
|---|---|
| 公開URL | **https://jic-events.pages.dev**（2026-11-13〜 Cloudflare Pages）。旧 https://jic-events.netlify.app は netlify.toml で新URLへ301転送（Netlifyの無料枠が復活する11/24以降に有効） |
| リポジトリ | https://github.com/kiwijic123-stack/jic-events （branch: main） |
| 公開方法 | Cloudflare Pages（アカウント kiwi.jic.123@gmail.com／プロジェクト jic-events）。mainにpushすると自動公開（ビルドなし・出力dir=/）。無料で月500ビルド。GitHub側はCloudflareアプリを jic-events だけに許可 |
| 申込通知先 | kiwi.jic.123@gmail.com（Googleフォーム「JICイベントカレンダー 受付（サイトから自動送信）」のメール通知。下の「サイト内フォームの送信先」参照） |
| 構成 | index.html 1枚（自己完結・JSレンダリング）＋ /soudan/ ＋ /camp/ ＋ チラシ画像jpg |

## 絶対に守ること

1. **push前に必ずjsdomでテストする**（過去にモーダル表示バグ・ボタン不動作で手戻りあり）
2. **日付・曜日は必ずコードでファクトチェック**（`new Date()`で曜日算出。2027年1月などの年またぎ注意）
3. **受講生の個人名・クラス用Zoomリンクをこのサイトに載せない**（スプレッドシートには載っているが公開禁止。公開してよいZoomは下記3つのみ）
4. **手動アップロード（zipドラッグ等）はしない**。更新はgit pushのみ（Cloudflareが自動公開）
5. （旧）Netlifyフォームは使っていない。Netlifyは無料枠（月20デプロイ）で止まったため引っ越した。netlify.toml は旧URL転送用なので消さない
6. 公開確認は https://jic-events.pages.dev をcurl/WebFetchで見る（反映は1〜2分）

## 更新の流れ

```bash
cd /home/claude/jic-events   # なければ git clone https://github.com/kiwijic123-stack/jic-events
git pull
# index.html を編集（EVENTSやCOURSES配列が主な編集対象）
# jsdomでテスト（NODE_PATH=/home/claude/node_modules node test.js）
git add -A
git -c user.name="JIC Events" -c user.email="kiwi.jic.123@gmail.com" commit -m "..."
git push origin HEAD:main
# 1〜2分でNetlifyが自動公開
```

jsdomテストの定型：`win.Element.prototype.scrollIntoView=function(){}`、`win.scrollTo=function(){}`、`win.fetch`をモック。クリックは`new win.MouseEvent("click",{bubbles:true})`。

## サイトの構造（2026-11-04時点）

- **タブ別ページ構成（SPA）**：トップ＝カラーバー型メニュー（Open Session／JIC受講生交流会／ざっくばらんお話会／説明会・個別相談／講座スケジュール／異文化理解研修）。ハッシュ`#v=`連動でスマホの戻るボタン対応。全ページ末尾に「次回のイベント」カード（直近イベントの申込カード）を常時表示
- **イベントは`EVENTS`配列**：`date`が今日以降＝募集中、過去＝終了済みに自動振り分け。手動で移動する必要なし
- **講座日程は`COURSES`配列**（1〜4期生の各クラス・全16回＋オリエン）
- **チラシ画像はタップで拡大**（lightbox。`image:`プロパティに900px幅・quality82のjpgを指定。PILで変換して repo直下に置く）
- カテゴリー色：Open Session=紺#24435C／交流会=黄金#C9A227／文法会=ワインレッド#8C2332（2026-11-13新設・日程はこれから）／お話会=緑#3F8A5A（2026-11-13 ピンク→緑）（表示名「JICひろば」＝ざっくばらんお話会・Café&Bar・わたしらしいセミナーをまとめる枠。EVENTSのcategoryは"お話会"のまま、メニューとタグだけJICひろばと表示）／説明会=緑#5F9440／講座=#3D6FA6／研修=#7A8087
- トップのメニュー（2026-11-13〜）：Open Session／JIC受講生（黄）／文法会／JICひろば。「JIC受講生」を押すと2タブ（講座スケジュール＝#v=courses／受講生交流会＝#v=交流会）。受講生まわりは黄色で統一。「説明会・個別相談」タブはLINEのリッチメニューにあるので置かない（#v=説明会 は生きている）。代わりにヘッダー右に常時表示の「無料個別相談 随時受付中」ボタン（soudan/へ）
- 異文化理解研修（JIC Camp 2026）は終了。メニューから外し、トップ最下部「終了した研修」に小さく残す（「くわしく見る」で#v=campの研修ページ）

## EVENTSのプロパティとボタンの出し分け（優先順）

1. `zoom`＋category"交流会" → 受講生限定モーダル（名前＋期を記入→Zoom表示。一般参加不可の注意文あり）
2. `ask: true` → **サイト内申込モーダル**（お名前・メール・お悩み必須 → Netlifyフォーム`event-entry`でJICメールに通知 → `zoom`があれば参加URL自動表示、なければ「追ってメールでご案内」表示）。お話会・説明会用
3. `reserve: true` → 予約モーダル（姓名・メール → `event-booking`）
4. `zoom`のみ → 「参加URLを見る」ポップ
5. `link`＋`afterLink` → 振り返りシート確認ポップ経由でGoogleフォームへ（Open Session・ざっくばらんお話会。終了後のカードには「振り返りシートを書く」ボタンが残る）
6. `link`のみ → 外部リンクボタン（`linkLabel`でラベル指定）
- `deadline: "11月13日（金）"` → カードのタイトル上に「〆 お申込みは◯◯まで」の帯（募集中のときだけ）
- `alwaysOpen: true` → 日付葉が「随時受付中」（無料個別相談 → /soudan/ へリンク）

## 公開してよいZoom

※Open SessionのZoomはサイトに載せない運用（Googleフォームの確認メッセージで自動案内。URL自体はスプレッドシート「スケジュール・Zoom管理」の『JIC Open Session URL』列にある。このファイルはNetlifyで公開されるため、ここにも書かない）

- **受講生交流会**：https://us06web.zoom.us/j/85287609476?pwd=jReYvMz3NvSocx4w9VYaiypk5HczhH.1 ／ ID 852 8760 9476 ／ パス 015436
- **ざっくばらんお話会**（2026-11-05〜）：https://us06web.zoom.us/j/88350567502?pwd=nCl9caxwFLYCDdcam1VrJDNPHewuBy.1 ／ ID 883 5056 7502 ／ パス 645373

## Googleフォーム（運用者側で設定済み・サイトからリンク）

- Open Session申込：1FAIpQLSdCXBwgwCq3UYV7C7KLxOC52AY4gM3Mhik0PiTcPBP7oAkNgA（確認メッセージでZoom案内）
- Open Session振り返りシート：1FAIpQLSeYJC-mzvyECS9b-zfpFlDxvwLt-OfqlIr03Ct4p0Bv76KF5A
- ざっくばらんお話会 申込（使い回し）：1FAIpQLScDccNIn6fhKZQvkRjJgJZljSf925XePuJNvlAneJl_9XeNJg（確認メッセージでZoom案内）
- ざっくばらんお話会 振り返りシート（使い回し）：1FAIpQLSdQMSeHFh2gCY2OI7--Qon5fN4V6vPr28tBdTNvewkdbMT43g
- 2つとも「回答を1回に制限」オフ・「回答の編集」オフ。参加者には「イベントカレンダーを見てね」だけで済む導線（申込も振り返りもサイトのカードから）

## サイト内フォームの送信先（2026-11-13〜 Googleフォームに移行）

- サイト内の申込（個別相談・Café&Bar等の予約・お話会/説明会の申込・交流会・研修）は、すべて `jicSend()` で JICのGoogleフォーム「JICイベントカレンダー 受付（サイトから自動送信）」へ送る（kiwi.jic.123のドライブ／編集ID 1u5w31YPZLaBcbCcchZadAACkHhvw76nnnplvKpSKXrw／公開ID 1FAIpQLSeZ9z8ZKJHpCVT4Hm2pO8Cskodo9zQRMs-2DUvcO8JJUUwehQ）。回答はフォームに貯まり、新しい回答はJICのメールに通知
- 欄：受付の種類 entry.129651137／イベント名（＋開催日時）entry.2103293136／お名前 entry.666709429／メールアドレス entry.441870969／ご相談内容・その他 entry.679977609（上記以外の項目を「項目名：値」で改行区切り）
- Googleフォームは質問名に「日時」などを入れると自動で日付型になるので注意
- 新しいサイト内フォームを作るときは、form-name を `jicSend` の KIND に足すだけでよい

## 旧：Netlifyフォーム名（Netlify時代。hiddenフォームは残っているが使っていない）

event-booking／camp-booking／kouryu-entry／soudan-booking（/soudan/）／event-entry（お話会・説明会の申込）
※新フォームを増やすときは、検出用のhidden静的フォームを置いてからpush

## 運用ルール（ユーザーの意向）

- スケジュール更新はスプレッドシート（受講生日程・スケジュール/Zoom管理）を渡されるので、**ダブルチェック体制**（機械抽出＋逆照合）で反映
- Open Sessionのナンバリングは2026年9月〜が第1回（8月分はお試し期間）
- チラシをもらったら該当イベントに画像を付け、タイトル・文言もチラシ表記に合わせる
- 交流会は受講生限定（一般参加不可の注意書きを維持）
- 更新したら「1〜2分で反映」と伝え、変更点を簡潔に報告

## 未解決・保留

- JIC Café&Bar（11/11）の参加Zoom URLが未登録（申込完了画面は「追ってメールでご案内」表示中）。URLが来たら`zoom`を追加すれば自動表示に切り替わる
- Café&Barのチラシは「公式LINEから申込」表記だが、サイトはサイト内フォーム受付。どちらに寄せるかユーザーに確認中
- Netlifyに重複サイト「benevolent-queijadas-1e8c3b」が残っている（削除は任意）
