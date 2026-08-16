# 自分で使う用の Slidev テンプレート

このままでは GitHub Pages にデプロイされないので以下の手順が必要。

- Settings > Pages > Build and deployment の「Source」を GitHub Actions に
- `bun install` を実行して、生成された `bun.lock` をコミットしてプッシュ
