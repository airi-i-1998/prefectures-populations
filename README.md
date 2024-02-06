# prefectures-populations

This template should help get you started developing with Vue 3 in Vite.

## Recommended IDE Setup

[VSCode](https://code.visualstudio.com/) + [Volar](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur) + [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin).

## Customize configuration

See [Vite Configuration Reference](https://vitejs.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```

### RESAS(地域経済分析システム) APIから必要なデータのAPI Keyを発行方法
①RESAS(地域経済分析システム) APIに利用登録し、API Keyを取得
②ローカル環境に`.env`作成
③取得したAPI Keyを貼り付ける
```
VITE_APP_RESAS_API_KEY="your_api_key"
```

### 参考資料
https://opendata.resas-portal.go.jp
※今回は、「都道府県一覧」APIと「人口構成API」を取得し、都道府県別の総人口推移折れ線グラフを作成いたしました。
