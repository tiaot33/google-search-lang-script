# google-search-lang-script

基于 Tampermonkey 的 Google 搜索结果语言过滤脚本。

脚本会在 Google 搜索结果页面上，根据用户配置的语言列表自动为 URL 添加或更新 `lr=lang_x|lang_y` 参数，例如：

- 标准搜索：`https://www.google.com/search?q=artificial+intelligence`
- 限定为英文和简体中文：`https://www.google.com/search?q=artificial+intelligence&lr=lang_en|lang_zh-CN`

## 功能概览

- 默认启用英语搜索语言（`lang_en`），即在未配置时自动对搜索结果使用英文过滤。
- 通过 Tampermonkey 菜单打开配置界面，勾选要启用的目标语言（例如 `lang_en`, `lang_zh-CN`）。
- 自动在 Google 搜索结果 URL 中应用或更新 `lr` 参数，不改变原始搜索关键词及其它查询参数。

## 安装

### 方式一：直接安装（推荐）

1. 安装 [Tampermonkey](https://www.tampermonkey.net/)（或兼容的用户脚本管理器）。
2. 点击下方链接直接安装脚本：
   - **[安装 Google Search Language Filter](https://raw.githubusercontent.com/tiaot33/google-search-lang-script/main/google-search-language-filter.user.js)**
3. 在 Tampermonkey 安装确认页面点击"安装"按钮。

### 方式二：手动安装

1. 安装 Tampermonkey（或兼容的用户脚本管理器）。
2. 在 Tampermonkey 中创建一个新脚本。
3. 将仓库根目录下 `google-search-language-filter.user.js` 的全部内容复制到新脚本中并保存。

### 自动更新

脚本支持自动更新功能：

- **更新检查**：脚本使用 `google-search-language-filter.meta.js` 文件进行版本检查
- **自动下载**：发现新版本时，Tampermonkey 会自动下载并安装更新
- **手动更新**：也可以在 Tampermonkey 管理面板中手动检查更新

脚本的匹配范围涵盖所有 Google 国际域名（如 `.com`、`.cn`、`.jp` 等），仅在 Google 搜索结果页（`/search` 路径）上执行语言过滤逻辑。

## 使用说明

### 配置搜索语言

1. 在浏览器工具栏中打开 Tampermonkey 菜单。
2. 找到该脚本（Google Search Language Filter）。
3. 点击“配置 Google 搜索语言过滤”菜单项。
4. 在弹出的配置窗口中勾选需要启用的语言。窗口顶部提供了一个搜索输入框，可以按“语言名称或代码”动态过滤列表；脚本内置了 `languagecodes.csv` 中列出的全部语言，例如：
   - English (`lang_en`)
   - Chinese (simplified) (`lang_zh-CN`)
   - Chinese (traditional) (`lang_zh-TW`)
   - Japanese (`lang_ja`)
   - Korean (`lang_ko`)
   - Spanish (`lang_es`)
   - French (`lang_fr`)
   - German (`lang_de`)
   - Russian (`lang_ru`)
   - Portuguese (`lang_pt`)
5. 点击“保存”后，新的语言组合会被持久化并应用于后续的 Google 搜索结果。

说明：

- 初次安装或尚未配置时，脚本默认启用 `lang_en`。
- 若在配置界面中未勾选任何语言，则脚本不会对 `lr` 参数做任何修改，相当于不启用语言过滤。

### 行为说明

- 当你在 `https://www.google.com/search` 上输入关键词并触发搜索时，脚本会：
  - 读取当前配置的语言列表（如 `lang_en`, `lang_zh-CN`）。
  - 生成 `lr` 参数值，例如 `lang_en|lang_zh-CN`。
  - 如果当前 URL 中没有 `lr`，则追加该参数；
  - 如果当前 URL 中已有 `lr` 但值与配置不一致，则用配置值覆盖；
  - 保持 `q`, `start`, `source` 等其它参数不变。
- 当 URL 中的 `lr` 与当前配置一致时，脚本不会触发额外跳转，避免循环重定向。

## 项目文件说明

仓库包含以下主要文件：

- **`google-search-language-filter.user.js`** - 完整的用户脚本文件，包含所有功能代码
- **`google-search-language-filter.meta.js`** - 元数据文件，仅包含脚本头部信息，用于自动更新检查
- **`languagecodes.csv`** - 语言代码参考表，列出所有支持的语言及其对应代码
- **`README.md`** - 项目说明文档

> **注意**：安装时只需使用 `.user.js` 文件。`.meta.js` 文件由 Tampermonkey 自动调用进行更新检查。

## 语言代码参考（languagecodes.csv）

仓库根目录的 `languagecodes.csv` 提供了"语言名称 → 语言代码 → Criterion ID"的完整对照表，例如：

```text
Language name,Language code,Criterion ID
English,en,1000
Chinese (simplified),zh_CN,1017
Chinese (traditional),zh_TW,1018
...
```

Google 的搜索语言过滤参数使用 `lr=lang_<Language code>` 的格式，其中：

- `Language code` 来自 `languagecodes.csv` 的第二列；
- `_` 通常在 `lr` 参数中写作 `-`，例如：
  - `en` → `lang_en`
  - `zh_CN` → `lang_zh-CN`
  - `zh_TW` → `lang_zh-TW`

你可以基于 `languagecodes.csv` 找到目标语言的 `Language code`，并推导出对应的 `lr` 参数值，从而确认脚本配置是否符合预期。

## 手动测试场景

以下为推荐的手动验证步骤，以确保脚本行为符合预期：

1. 默认英文过滤
   - 确保脚本为首次安装或在配置中只勾选 `English (lang_en)`。
   - 打开 `https://www.google.com/search?q=artificial+intelligence`。
   - 预期：最终地址包含 `lr=lang_en`。
2. 多语言过滤（英文 + 简体中文）
   - 在配置界面中勾选 `English (lang_en)` 和 `Chinese (Simplified) (lang_zh-CN)`。
   - 再次访问 `https://www.google.com/search?q=artificial+intelligence`。
   - 预期：最终地址包含 `lr=lang_en|lang_zh-CN`。
3. 更新已有 lr 参数
   - 在配置界面中保持 `lang_en|lang_zh-CN`。
   - 访问 `https://www.google.com/search?q=artificial+intelligence&lr=lang_fr`。
   - 预期：URL 中的 `lr` 被更新为 `lang_en|lang_zh-CN`，其它参数保持不变。
