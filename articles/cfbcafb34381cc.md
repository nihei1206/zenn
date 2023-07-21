---
title: "社内専用のChatGPTをChatbotUIを使って実現してみた"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ChatGPT"]
published: false
---

# こんにちは
こんにちは、ベトナムの日系IT企業で、PMOをしています。Niheiです。
今日は、日本がおやすみでお客さんとのMTGないので、ChatGPTに関する記事をサラッと書いていみます。
( ベトナムは日本より休日が少ないので、準委任契約での開発をするには、日本よりいいですよね。開発の[お問い合わせ](https://www.vitalify.asia/)お待ちしてます 笑 )

# 社内ChatGPTを構築しよう
さて、社内専用のChatGPTを使うことで、社外にデータが漏れないようにしたいですよね。

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
( 他のバージョンでもいけると思いますが、記載なかったので、18をいれました。 )
2. [chatbot-ui](https://github.com/mckaywrigley/chatbot-ui)をローカル環境にcloneします。
3. ルートに、環境変数のファイルを設置します。
```
OPENAI_API_KEY=xxxx
OPENAI_ORGANIZATION=org-xxxxx
DEFAULT_MODEL=GPT3.5-turbo
```
4. npm run devをすると起動します。
5. ローカルでの動作が確認できたら、Vercelを使って、デプロイしましょう！
