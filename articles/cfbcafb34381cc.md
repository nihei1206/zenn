---
title: "社内専用のChatGPTをChatbotUIを使って実現してみた"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ChatGPT","Vercel","Nextjs"]
published: false
---

# こんにちは
こんにちは、ベトナムの日系IT企業で、PMOをしています。Niheiです。
今日は、日本がおやすみでお客さんとのMTGないので、ChatGPTに関する記事をサラッと書いていみます。
( ベトナムは日本より休日が少ないので、準委任契約での開発をするには、日本よりいいですよね。開発の[お問い合わせ](https://www.vitalify.asia/)お待ちしてます 笑 )

ベトナムでは、ChatGPTに登録際に、ベトナムの電話番号を使って認証することができないため、正規の方法でChatGPTのアカウントを作ることができず、他社がUIを提供しているツールを使っている方が多いのが現状です。（または、グレーな手段で一時的なSMS認証をしたり、他人のアカウントを借りたり、、、なので、世界でもChatGPTアカウントのハッキングにあっているユーザーが多いことでも、ニュースになっていたりしました。怖いですねぇ〜〜〜。）

# 社内ChatGPTを構築しよう
そこで、社内専用のChatGPTを使うことで、社外にデータが漏れないようにしたいですよね。

ChatGPTに入力したデータを、ChatGPTの学習に利用されないようにするには、前回ご紹介した通り、[オプトアウト申請](https://zenn.dev/vfa/articles/7b18deeb385e41)をする必要がありますし、さらに厳重にやるとなると、[Microsoft OpenAI Service](https://azure.microsoft.com/en-us/products/cognitive-services/openai-service)からAPI Keyを発行することで、もっとプライバシーを保護できます。

今回は、簡単のため、Azureではなく、OpenAIからAPI keyを発行してやってみました。また、ローカル環境でやったあとに、Vercelを使ってデプロイしました。
Dockerを使う方法も提供されていますね。

## 使ったOSS

OSSとして、提供されている[chatbot-ui](https://github.com/mckaywrigley/chatbot-ui)を使いました。

## 環境
```
Node.js v18.16.1
```

## 手順

1. nvmを使って、v18を入れます。
( 他のバージョンでもいけると思いますが、記載なかったので、最新の18をいれました。 )
2. [chatbot-ui](https://github.com/mckaywrigley/chatbot-ui)をローカル環境にcloneします。
3. 自分のGithubアカウントにレポジトリを作成し、自分のGithubで管理できるようにしましょう。
4. ルートに、env.localという名前で、環境変数のファイルを設置します。
他にも環境変数はありますが、中身はこれをコピペすれば大丈夫です。
API keyは、[こちら](https://platform.openai.com/account/api-keys)から取得しましょう。
Microsoft Azure OpenAI ServiceのAPIも使えますが、今回は、OpenAIのAPIを使いました。
```plain:env.local
OPENAI_API_KEY=xxxx
OPENAI_ORGANIZATION=org-xxxxx
DEFAULT_MODEL=GPT3.5-turbo
```

:::message
OPENAI_ORGANIZATIONに、請求先が紐づきますので、複数の組織に所属しているアカウントで行う方は、OPENAI_ORGANIZATIONを[こちら](https://platform.openai.com/account/org-settings)から取得して、設定してください。
:::

4. npm run devをして、localhost:3000にアクセスすると、起動します。
5. ローカルでの動作が確認できたら、Vercelを使って、デプロイしましょう！
6. [Vercel](https://vercel.com/)にアクセス
7. この画面で、Create New Projectをクリック
![](https://storage.googleapis.com/zenn-user-upload/9e6eef21c00a-20230721.png)
8. この画面で、Githubアカウントと接続し、デプロイするレポジトリを選択。importをクリック
![](https://storage.googleapis.com/zenn-user-upload/b7e0d10ce1f0-20230721.png)
9. この画面になって、自動でデプロイ作業が始まるので、終わるのを待ちます。
![](https://storage.googleapis.com/zenn-user-upload/2dd397be05f5-20230721.png)
10. この画面になったら成功！画面の画像をクリックすると、専用ChatGPTのURLにジャンプします。
![](https://storage.googleapis.com/zenn-user-upload/35048a1a241c-20230721.png)
11. できましたね！
![](https://storage.googleapis.com/zenn-user-upload/fa795e147e86-20230721.png)

Basic認証を追加したかったらそのようにできるみたいなので、そこらへんはご自身で設定してください。

# 注意点
:::message
- URLを知っていれば誰でもアクセスできてしまうので、そこらへんの制限は各社のご状況ごとに実施してください。
- [Usage limits](https://platform.openai.com/account/billing/limits)から、一定の金額以上は使えないように設定しておくと、使われすぎを防止できるのでおすすめです。
:::

# まとめ
エンジニアじゃない私が、ここまで15分ほどでできてしまいました。(むしろ、Githubとローカルレポジトリとの接続、SSHとか、そこらへんで想定外の時間がかかってしまいました笑)

社内専用ChatGPTを構築し、社内に共有することで、社外への情報漏洩リスクも抑えることができますし、特にベトナムなど、ChatGPTを正規の方法で使えない国で、スタッフが外部の怪しいツールを使ってしまうことを防止できます。

社内専用ChatGPTの構築はもちろん、ChatGPTを業務にどのように活かせばいいか悩んでいる企業様向けの相談サービスをご提供しておりますので、ご興味あればご連絡ください。

# 余談
ちなみに、このChatbot-uiは、多言語対応しており、URLの最後に/jaとすると日本語になりますし、/viとするとベトナム語になります。
