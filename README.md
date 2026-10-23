# passenger-content-distribute

一个 Codex 内容分发编排 Skill，名称为 `content-distribute`。负责调用现有 Skills、安排顺序并验收；不包含另一套生图、排版或发布实现。

## 三种模式

| 模式 | 输入与结果 | 默认远端行为 |
| --- | --- | --- |
| A：公众号长文 | 微信双裁切＋X 独立封面；可选公众号排版；X 原生文章改写 | 先公众号草稿验收，再 X 文章草稿 |
| B：公众号贴图转小红书 | 复用有序图片，重写标题、正文与 5–10 标签 | 仅交付素材 |
| C：仅补封面 | 默认微信双裁切＋X 封面；可限定单端 | 不写远端 |

**两路封面必须从第一次规划起同时存在。** 工具支持时同轮并发生成；仅支持串行时先登记两路并保存两份提示词，记录原因后串行执行，不能完成微信后才想起 X。

微信默认 `wechat-dual-crop-master`：2010×600 母版，左 1410×600 横版、右 600×600 方版，两边各有完整标题与主体。X 依据当前 `$punk-cover` 文档独立构图，当前为 5:2。

## 安装

仓库根目录就是一个完整 Skill，不另设 `skills/` 副本。复制整个仓库目录到 `~/.codex/skills/content-distribute`：

```bash
git clone https://github.com/Passenger-0522/passenger-content-distribute.git
mkdir -p "$HOME/.codex/skills"
cp -R passenger-content-distribute "$HOME/.codex/skills/content-distribute"
```

上面的复制命令适用于目标尚不存在时；已有同名 Skill 时先检查和备份，不直接嵌套复制。也可将已克隆工作副本链接到安装位置：

```bash
ln -s "$PWD/passenger-content-distribute" "$HOME/.codex/skills/content-distribute"
```

在 Codex 的新会话中使用 `$content-distribute`；已有会话未刷新清单时可先明确指向安装目录的 `SKILL.md`。

## 依赖

依赖由使用者单独安装；本仓库不复制其实现、不携带账号凭据。依赖名称和用途如下，实际位置以 Codex 的可用技能清单为准。

| 依赖 | 调用场景 |
| --- | --- |
| `$punk-cover` ＋ `~/.codex/styles` | A／C 的微信与 X 封面，需完整共享风格库 |
| `$gzh-design` | 可选公众号 HTML、复制预览和校验 |
| `$xhs-note-creator` | B 的文案；需要新卡片时才使用渲染步骤 |
| `$dbs-xhs-title` | B 的公式标题候选与编号溯源 |
| `$baoyu-xhs-images` | 按需新增小红书／微信贴图卡片 |
| 文颜 CLI 的已核实草稿能力，或公众号官方草稿 API | 推送公众号草稿 |
| 已连接 X 插件的 `article_create_draft` | 创建 X Articles 文章草稿并附封面 |

图像后端由现有生图 Skills 选择。X 插件或公众号工具不在当前运行时可用清单里时，只交付本地稿并列出缺口，不宣称已推送。依赖安装位置不等于授权已完成。

## 用法示例

```text
$content-distribute 分发这篇 ./article.md：封面风格 auto，公众号用摸鱼绿排版，先推公众号草稿，再建 X 文章草稿；不用确认。

$content-distribute 公众号已经发完这篇，链接是 <我的文章链接>。同步到 X 文章草稿，封面风格 auto。

$content-distribute 把 ./wechat-cards/ 按文件顺序转成小红书素材，原图复用，重写标题、正文和标签。

$content-distribute 给 ./article.md 补封面，微信双裁切和 X 都要，风格 auto，不用确认。

$content-distribute 只补 X 封面，使用我指定的风格，不推草稿。
```

“一键分发”会进入 A；“贴图发小红书”默认进入 B 并产出素材。只有明确要求发布到小红书才进入发布流程；公开可见还需明确公开授权。仅准备素材可直说“只出本地包，不推草稿”。

## 交付与状态

每次在项目 `content-distribute/{slug}/{run-id}/` 保存 `run.json`、`delivery.md` 和真实产物路径。按微信、X、小红书逐端报告素材、草稿 ID／链接、回读证据和缺口，未请求端标记 `not_requested`。

- `completed`：约定范围内素材和要求的草稿验收都通过。
- `partial`：已有有效产物，但仍缺封面、依赖、推稿或回读证据。
- `blocked`：前置条件阻断，尚无可交付结果。

HTML 结构校验、视觉检查、编辑器粘贴和远端回读分别记录；提示词不等于图片，草稿不等于发布。创建请求超时先核查，不盲目重试产生重复草稿。

## 权限与隐私

公众号仅草稿，不群发、不公开发布；X 仅文章草稿，不擅自公开发帖；小红书默认不调用发布接口，包含仅自己可见的发布。只使用现有安全配置，不把 AppSecret、token、Cookie 或 `.env` 写入仓库、提示词和日志。

不登录或使用 Amazon 账号，不启动或操作紫鸟，不推断 Amazon API 远端写入授权。用户全局规则持续适用。

## 目录

```text
SKILL.md                 编排入口与检查清单
agents/openai.yaml        Codex 展示与触发元数据
references/dependencies.md 依赖调用契约
references/draft-push.md  草稿顺序、验收与重跑
references/delivery.md    运行记录与三端交付
LICENSE                  MIT
.gitignore               凭据和用户输出排除
```

这是编排说明包；安装不会自动执行内容推送，也不表示已在使用者账号测试过远端草稿链路。
