---
description: "Tailwind CSS v4+ の Vite プロジェクト開発ガイドライン"
applyTo: "**/vite.config.ts, **/*.css, **/*.tsx"
---

# Tailwind CSS v4+ と Vite のインストール

公式 Vite プラグインを使用した Tailwind CSS バージョン 4 以降のインストールと設定手順です。
Tailwind CSS v4 では、ほとんどの場合 PostCSS 設定や tailwind.config.js が不要になる、簡素化されたセットアップが導入されました。

## Tailwind CSS v4 の主な変更点

- Vite プラグイン使用時は **PostCSS 設定が不要**
- **tailwind.config.js が不要** - 設定は CSS で行う
- **新しい @tailwindcss/vite プラグイン** が PostCSS ベースのアプローチに置き換わる
- `@theme` ディレクティブを使用した **CSS ファースト設定**
- **自動コンテンツ検出** - content パスの指定が不要

## インストール手順

### ステップ 1: 依存関係のインストール

`tailwindcss` と `@tailwindcss/vite` プラグインをインストール:

```bash
npm install tailwindcss @tailwindcss/vite
```

### ステップ 2: Vite プラグインの設定

Vite 設定ファイルに `@tailwindcss/vite` プラグインを追加:

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [
    tailwindcss(),
  ],
});
```

Vite を使用した React プロジェクトの場合:

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [
    react(),
    tailwindcss(),
  ],
});
```

### ステップ 3: Tailwind CSS のインポート

メイン CSS ファイル (例: `src/index.css` または `src/App.css`) に Tailwind CSS のインポートを追加:

```css
@import "tailwindcss";
```

### ステップ 4: エントリポイントでの CSS インポートの確認

アプリケーションのエントリポイントでメイン CSS ファイルがインポートされていることを確認:

```typescript
// src/main.tsx または src/main.ts
import "./index.css";
```

### ステップ 5: 開発サーバーの起動

開発サーバーを実行してインストールを確認:

```bash
npm run dev
```

## Tailwind v4 でやってはいけないこと

### tailwind.config.js を作成しない

Tailwind v4 は CSS ファースト設定を使用します。特定のレガシー要件がない限り、`tailwind.config.js` ファイルを作成しないでください。

```javascript
// ❌ Tailwind v4 では不要
module.exports = {
  content: ["./src/**/*.{js,ts,jsx,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

### Tailwind 用の postcss.config.js を作成しない

`@tailwindcss/vite` プラグインを使用する場合、Tailwind 用の PostCSS 設定は不要です。

```javascript
// ❌ @tailwindcss/vite 使用時は不要
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

### 古いディレクティブを使用しない

古い `@tailwind` ディレクティブは単一のインポートに置き換わりました:

```css
/* ❌ 旧式 - Tailwind v4 では使用しない */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* ✅ 新式 - Tailwind v4 ではこちらを使用 */
@import "tailwindcss";
```

## CSS ファースト設定 (Tailwind v4)

### カスタムテーマ設定

CSS で `@theme` ディレクティブを使用してデザイントークンをカスタマイズ:

```css
@import "tailwindcss";

@theme {
  --color-primary: #3b82f6;
  --color-secondary: #64748b;
  --font-sans: "Inter", system-ui, sans-serif;
  --radius-lg: 0.75rem;
}
```

### カスタムユーティリティの追加

CSS で直接カスタムユーティリティを定義:

```css
@import "tailwindcss";

@utility content-auto {
  content-visibility: auto;
}

@utility scrollbar-hidden {
  scrollbar-width: none;
  &::-webkit-scrollbar {
    display: none;
  }
}
```

### カスタムバリアントの追加

CSS でカスタムバリアントを定義:

```css
@import "tailwindcss";

@variant hocus (&:hover, &:focus);
@variant group-hocus (:merge(.group):hover &, :merge(.group):focus &);
```

## 確認チェックリスト

インストール後、以下を確認:

- [ ] `tailwindcss` と `@tailwindcss/vite` が `package.json` の dependencies に存在
- [ ] `vite.config.ts` に `tailwindcss()` プラグインが含まれている
- [ ] メイン CSS ファイルに `@import "tailwindcss";` が含まれている
- [ ] CSS ファイルがアプリケーションのエントリポイントでインポートされている
- [ ] 開発サーバーがエラーなしで実行される
- [ ] Tailwind ユーティリティクラス (例: `text-blue-500`、`p-4`) が正しくレンダリングされる

## 使用例

シンプルなコンポーネントでインストールをテスト:

```tsx
export function TestComponent() {
  return (
    <div className="min-h-screen bg-gray-100 flex items-center justify-center">
      <h1 className="text-3xl font-bold text-blue-600 underline">
        Hello, Tailwind CSS v4!
      </h1>
    </div>
  );
}
```

## トラブルシューティング

### スタイルが適用されない

1. CSS インポート文が `@import "tailwindcss";` であることを確認 (旧ディレクティブではない)
2. CSS ファイルがエントリポイントでインポートされていることを確認
3. Vite 設定に `tailwindcss()` プラグインが含まれていることを確認
4. Vite キャッシュをクリア: `rm -rf node_modules/.vite && npm run dev`

### プラグインが見つからないエラー

「Cannot find module '@tailwindcss/vite'」と表示される場合:

```bash
npm install @tailwindcss/vite
```

### TypeScript エラー

TypeScript が Vite プラグインの型を見つけられない場合、正しいインポートを確認:

```typescript
import tailwindcss from "@tailwindcss/vite";
```

## Tailwind v3 からの移行

Tailwind v3 から移行する場合:

1. `tailwind.config.js` を削除 (カスタマイズは CSS の `@theme` に移動)
2. `postcss.config.js` を削除 (Tailwind のみに使用していた場合)
3. 古いパッケージをアンインストール: `npm uninstall postcss autoprefixer`
4. 新しいパッケージをインストール: `npm install tailwindcss @tailwindcss/vite`
5. `@tailwind` ディレクティブを `@import "tailwindcss";` に置き換え
6. Vite 設定を `@tailwindcss/vite` プラグインを使用するように更新

## リファレンス

- 公式ドキュメント: [https://tailwindcss.com/docs/installation/using-vite](https://tailwindcss.com/docs/installation/using-vite)
- Tailwind CSS v4 アップグレードガイド: [https://tailwindcss.com/docs/upgrade-guide](https://tailwindcss.com/docs/upgrade-guide)
