# mdbook使用记录
## 使得mdbook在发布后支持数学公式
### 1. 根目录的book.toml中注释或者删除math相关的所有
### 2. 项目根目录与src平级的目录处， 新建theme目录，theme目录新建文件head.hbs,这个文件会被mdbook自动加载。
mdbook内容如下:
```js
<script>
  window.MathJax = {
    tex: {inlineMath: [['$', '$'], ['\\(', '\\)']]}
  };
</script>
<script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
<script>
  function renderMath() {
    if (window.MathJax) {
      MathJax.typesetPromise();
    }
  }

  // 页面初次加载
  document.addEventListener("DOMContentLoaded", renderMath);

  // mdBook 切换章节时
  document.addEventListener("DOMContentLoaded", () => {
    const observer = new MutationObserver(renderMath);
    observer.observe(document.querySelector("#content"), { childList: true, subtree: true });
  });
</script>
```

## mdbook 结合github action发布太慢优化
观察发现，github每次打包都要重新便宜mdbook，导致花费时间太长，所以优化mdbook.yml直接用已经打包的二进制包
样例：
```yml
name: Deploy mdBook site to Pages

on:
  push:
    branches: ["master"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # 直接下载你打好的 Linux 版 mdBook
      - name: Install mdBook (Chinese search fork)
        run: |
          curl -L -o mdbook https://github.com/zhangyinyuan/mdBook/releases/download/auto-build/mdbook-linux
          chmod +x mdbook
          sudo mv mdbook /usr/local/bin/

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Build with mdBook
        run: mdbook build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./book  # 注意自己大号包之后的文件夹的名称， 由book.toml文件中的build-dir属性决定

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```
