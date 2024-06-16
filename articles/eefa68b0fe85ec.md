---
title: "GASとChatGPTで構成するAIラジオ"
emoji: "📻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [GAS,ChatGPT,tts,OpenAI,個人開発]
published: true
---

## はじめに
こんにちは、ベトナムでPMとPMOと生成AIアドバイザーをしているmotaです。
最近 GPTのJSON Responseを使ってなにかやりたいなぁと思っていたところ、こんな投稿を見つけました。

https://twitter.com/himara2/status/1791060681984495624

しかし、コストが1回$1と書かれておられまして、もっとなにか工夫をすればコストを下げられるのでは？と思い、個人的に試してみようかなとなりまして

そこで、zennに限らずに、かつAIにおすすめの記事をリコメンドさせるところからGPTにさせればいいのでは？と思いå、Google Apps Script(GAS,役割:js)とGoogle Drive(役割:storage)だけでAIラジオを作ってみたので、その話を書きます。
↓作ったのはこちら↓
https://www.youtube.com/playlist?list=PLlLMystd2WB0soKkXTflSJ56cLCutU5v_

## 技術構成

技術構成はこんな感じで、すべてGASとGPTだけで行いました。
![](https://storage.googleapis.com/zenn-user-upload/6809b00907d3-20240616.png)
簡単に説明しますと、
1. RSS(feed/atom)を公開しているニュースサイトの最新かつ今日の記事を複数取得してくる
2. タイトルだけを同時にGPT4oに与えてから、おすすめの数記事とそのURLをJSONでレスポンスさせる
3. Jina.aiにURLを投げてPlain Textで取得する
4. Plain Textの中からURLやコード・スペースを古典的手法(.replace)で削除をしてから、GPT3.5turboに要約させる
5. 要約結果をラジオ風に成形して、OpenAIのTest to Speechに話してもらう
6. Google Driveに保存し、音声ファイルのFile IDを使ってYouTube Data API V3を使ってYouTubeに投稿を設定する。
( 実装はしましたが、実はここはいま手動です。Spotifyに投稿するとYouTubeにもApple PodcastにもRSSで自動投稿できる機能が発覚したので、ならSpotifyにAPI経由でアップロードした方が楽じゃん。となっており、検討中です。 )

## Coding
Codingを一部公開しますと、こんな感じです。

まず GPT4oにResponse=json modeで、このような指示をします。
```text:Prompt
const gpt4Messages = [
    { 'role': 'system', 'content': 'ニュース記事のタイトルから、web/mobile/VRをシステム開発しているPM向けのおすすめを選びます。宣伝感のある記事は選ばずに、革新的な技術・AI・セキュリティ・プロジェクトマネジメントの観点から選んでください。' },
    { 'role': 'system', 'content': 'reply by json, japanese.  recommendations:[{ title:title , link:url },{ title:title , link:url }]' },
    { 'role': 'user', 'content': `以下のニュース記事の中からおすすめの${CHOICE_ARTICLE_NUM}つを選んでください:\n${articlesText}` }
  ];
```

そんで、各々にこのプロンプトでGPT3.5turboで要約をします。
```text:Prompt
const gpt35Messages = [
      { 'role': 'system', 'content': 'Summarise the article within 100 words in Japanese. {contents: summarised } by json' },
      { 'role': 'user', 'content': `:\n contents: ${cleanedAndTrimmedText}` }
    ];
```
なんやかんやこのような形でレスポンスが来ます。
```:要約後 & ラジオ生成前
[{"title":"VercelがPPRをNodeランタイムにした件からWebフロントエンドとエッジの動向に迫る",
"link":"https://zenn.dev/sumiren/articles/7886f5079cced0",
"summary":"VercelがWebサイトのエッジでのサーバーサイド処理を停止し、Next.jsにPPRを使用するとNodeランタイムで処理されることが話題となっている。エッジの利点やエッジの意味について解説された記事で、エッジロケーションやエッジランタイムについて詳しく説明がなされている。Vercelの動向についての考察や技術的な情報も含まれている。"},
{"title":"Google、ChromeOSをAndroidベースに　AI機能拡充へ",
"link":"https://www.itmedia.co.jp/news/articles/2406/14/news148.html",
"summary":"GoogleはChromeOSをAndroidベースにし、AI機能を拡充することを発表。これにより、ユーザーに迅速にサービスを提供できるようになるほか、ハードウェアとの互換性が向上する。GeminiNanoを通じてAI機能を展開する一方、ChromebookPlusではGoogleAIが利用可能。将来的には一般のChromebookでもGoogleAIが使えるようになる可能性がある。この取り組みはすでに開始されているが、一般ユーザー向けの提供までにはまだ時間がかかる見通し。"},
{"title":"組み込み機器でも生成AIが使える　日本発のアクセラレーター","link":"https://eetimes.itmedia.co.jp/ee/articles/2406/14/news021.html",
"summary":"日本のスタートアップ企業であるEdgeCortixが、エッジAI向けの新プラットフォーム「SAKURA-II」を発表しました。このプラットフォームは畳み込みニューラルネットワークやトランスフォーマーモデルを実装できる特徴があり、組み込み機器向けに生成AIを活用することが可能です。EdgeCortixはIP、ハードウェア、ソフトウェアを提供し、SAKURA-IIは既存のSAKURA-Iの後継製品となることが紹介されています。"},{"title":"根づき始めたメタバース。経済圏をけん引する意外なプレイヤー | Forbes JAPAN 公式サイト（フォーブス ジャパン）",
"link":"https://forbesjapan.com/articles/detail/71600",
"summary":"メタバースやVRがバズワードから脱し、日本でも広がりを見せている。VRChatのユーザー数が急成長し、国内外で様々なメタバース空間が生まれている。日本でも経済圏が形成され、3Dモデルの流通額が急増していることが明らかになっている。VRChatを中心に、日本のメタバース市場が急拡大しており、今後の成長が期待されている。"}]
```

それに対して、ラジオの原稿をこうやって生成しています。

```js:GAS
function generateRadioScript(summaryArray) {
  const today = new Date();
  const formattedDate = Utilities.formatDate(today, Session.getScriptTimeZone(), 'yyyy年MM月dd日');

  let script = `### みなさん、こんにちは。きょうは、${formattedDate}です\n\n。今日は最新のテクノロジーに関するニュースをお届けします。\n\n`;

  if (summaryArray.length === 1) {
    script += `今日のニュースは、${summaryArray[0].title}からです。${summaryArray[0].summary}\n\n`;
  } else {
    script += `最初のニュースは、${summaryArray[0].title}からです。${summaryArray[0].summary}\n\n`;
    for (let i = 1; i < summaryArray.length - 1; i++) {
      script += `次のニュースは、${summaryArray[i].title}からです。${summaryArray[i].summary}\n\n`;
    }
    script += `最後のニュースは、${summaryArray[summaryArray.length - 1].title}からです。${summaryArray[summaryArray.length - 1].summary}\n\n`;
  }

  script += "以上、本日のニュースでした。お聞きいただき、ありがとうございました。次回もお楽しみに。\n";

  return script;
}
```

## さいごに
これを応用すれば、どの国の言葉でもできそうでわくわくしています。
参考にさせていただいた方のようにお便りコーナーなどを作っても面白そうですが、力尽きたので、一旦ここで一区切りとさせていただきました。



## 参考記事
冒頭の方の記事を参考にさせていただきました。
https://zenn.dev/himara2/articles/db054d81b05d19

---
なんだかんだYouTubeの自動アップロードが一番大変でした。そのうち別で記事にしようと思います。
https://odyprograming.com/gas_youtube_upload_post_token/
