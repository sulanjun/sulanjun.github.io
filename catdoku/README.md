# 隐私政策

官网与协议统一放在 `docs/catdoku/`，再按平台分目录，每个平台目录自成一套，可以单独发布。

```text
docs/catdoku/
  index.html              协议总入口，列出各平台
  README.md
  openpaas/               OpenPaaS 平台目录
    index.html            英文协议
    zh-CN.html            简体中文协议
    policy.css
  google-play/            Google Play 平台目录
    index.html            英文官网
    privacypolicy.html    英文协议
    app-icon.png
    feature-graphic.png
```

OpenPaaS 协议由脚本生成；Google Play 官网与协议位于同一平台目录，其中协议为手工维护的英文页面。

新增游戏时，在 `config.mjs` 里复制一份平台清单并改 `SITE.gameSlug`，入口页会按游戏分组；
如果只有猫咪数独一个游戏，就只维护 `catdoku/` 这棵子树。

## 新增一个平台

1. 在 `scripts/privacy/config.mjs` 的 `PLATFORMS` 里加一条，字段含义：

   | 字段 | 说明 |
   | --- | --- |
   | `slug` | 目录名，例如 `google-play` |
   | `order` | 列表排序 |
   | `nameEn` / `nameZh` | 平台名 |
   | `kindEn` / `kindZh` | 载体说明，例如「Android 应用」 |
   | `ads` | 该版本是否展示广告 |
   | `adServiceEn` / `adServiceZh` | 广告由谁提供，决定「广告」章节措辞 |
   | `permissions` | 发行包实际声明的设备权限数组 |
   | `purchases` | 是否含内购 |
   | `locales` | 可选，输出语言；Google Play 仅发布英文 |
   | `manual` | 可选，`true` 表示页面手工维护，构建只更新入口列表 |

2. 执行 `node scripts/privacy/build.mjs`，会自动建 `docs/catdoku/<slug>/` 目录、生成页面，并把新平台加进 `docs/catdoku/index.html`。
3. 如果该平台有章节需要写得不一样（例如原生应用的权限清单），在 `sections-zh.mjs` / `sections-en.mjs` 里按 `platform.slug` 分支。
4. 在 `docs/privacy/README.md` 这页的目录树里补一行。

## 改文案

不要直接改 HTML，会被下次生成覆盖。源文件都在 `scripts/privacy/`：

- `config.mjs`：游戏名、开发者、邮箱、日期、适龄提示、平台能力。
- `sections-en.mjs` / `sections-zh.mjs`：章节正文，13 章。
- `policy.css`：样式，构建时复制到每个平台目录。
- `build.mjs`：生成脚本。

Google Play 的 `docs/catdoku/google-play/privacypolicy.html` 为手工维护页面，不在自动生成循环中覆盖；其入口仍由 `config.mjs` 和构建脚本统一维护。

```powershell
node scripts/privacy/build.mjs
```

没有独立 node 时：

```powershell
$env:ELECTRON_RUN_AS_NODE = '1'
& "<PinK安装路径>/PinK.exe" scripts/privacy/build.mjs
```

## 发布

平台要求可访问的 https 链接，`file://` 不行。把 `docs/catdoku` 整个目录传到静态托管，保持目录结构：

- 协议总入口：`https://<域名>/catdoku/`
- OpenPaaS：`https://<域名>/catdoku/openpaas/`
- 中文版：`https://<域名>/catdoku/openpaas/zh-CN.html`
- Google Play 官网：`https://<域名>/catdoku/google-play/`
- Google Play 协议：`https://<域名>/catdoku/google-play/privacypolicy.html`

游戏名称、主体、邮箱与页面保持一致：《猫咪数独》（Catdoku），王泽秋，wzeqiu@163.com。

## 写作边界

这份文件只写信息怎么被处理。以下内容不写进政策，改文案时不要加回来：

- 商业模式：收入来源、免费运营、广告变现。
- 玩法规则：激励视频什么时候出现、看完发什么奖、中途关闭会怎样、广告失败能否继续。
- 宣传语：游戏特色、设计理念、给玩家带来的感受。
- 文件自身说明：双语版本优先级、本政策的定位与作用。

## 与代码的对应关系

政策里的事实要能在代码里对上，改代码时同步检查：

| 政策表述 | 代码位置 |
| --- | --- |
| 无账号、无自建服务器、进度只存本地 | `assets/scripts/platform/Storage.ts` |
| 未声明任何设备权限 | `build/openpaas/manifest.json` 的 `permissions`（当前为空数组） |
| Google Play 权限与 AdMob App ID | `build-templates/native/engine/google-play/app/AndroidManifest.xml` |
| Google Play 广告、UMP 与 Firebase SDK | `build-templates/native/engine/google-play/app/build.gradle` |
| 广告服务处理设备信息、广告标识符、IP 与展示点击记录 | `assets/scripts/platform/Ads.ts` 与平台广告组件接入 |
| 不向广告服务提供玩家画像与游戏内行为 | `assets/scripts/game/CatdokuApp.ts` 的 `booster()` 只传广告位类型 |
| 适龄 8+ | `config.mjs` 的 `SITE.minAge`，同时要与平台后台的适龄提示一致 |

广告目前由 OpenPaaS 平台提供（`sud.createRewardedVideoAd`，`onClose` 判断是否看完）。
Google Play 版本使用 Google AdMob、UMP、Firebase Analytics 与 Crashlytics，因此单独的 Google Play 政策会披露这些服务。
Android 原生版本的 AdMob 适配器代码仍在仓库里，但小游戏包不加载它，因此 OpenPaaS 的政策里不出现 AdMob。
如果以后小游戏改用其他广告来源，需要同步改 `config.mjs` 的 `adService` 与「广告」章节。
