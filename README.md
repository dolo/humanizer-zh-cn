# Humanizer

Humanizer 用于自然改写中文和英文 AI 腔文本，保留作者的事实、观点、语气与读者对象。它面向人类读者，不承诺绕过 AI 检测器，也不凭几个词语判断作者身份。

本仓库是在 [blader/humanizer](https://github.com/blader/humanizer) 基础上改造的中英双语版本。原项目的 26 条英文规则得到保留，新增 24 条中文规则，以及语言路由、中文场景适配和事实校验。

**中文改写前：**
> 随着云计算技术的不断发展，缓存对系统性能提升具有重要意义。缓存不仅能减少重复查询、避免重复访问数据库，更能赋能业务流程，体现了现代架构的核心价值。这一点充分说明，缓存非常重要。

**中文改写后：**
> 缓存能减少重复查询，避免重复访问数据库。

改写不会为了让句子“具体”而编造数据、案例、来源、经历或观点。原文没有细节时，Humanizer 会保留诚实的概括、要求补充，或按规则删去没有依据的主张。

## 安装

安装后使用 `/humanizer`，也可以直接用自然语言要求 Agent 调用 Humanizer。

### Claude Code

```text
/plugin marketplace add dolo/humanizer-zh-cn
/plugin install humanizer@humanizer
```

插件要求 Claude Code 2.1.142 或更新版本。旧版本可运行：

```bash
npx skills add dolo/humanizer-zh-cn --global --agent claude-code
```

### Codex

```bash
npx skills add dolo/humanizer-zh-cn --global --agent codex
```

### pi

```bash
pi install git:github.com/dolo/humanizer-zh-cn
```

加 `-l` 可只安装到当前项目。`pi update --extensions` 用于更新。

### Claude.ai 和 Claude Desktop

下载整个仓库的 ZIP（Code → Download ZIP），再作为 Skill 上传。ZIP 必须包含 `references/`，因为中英文规则按需从这里加载。

### 其他 Agent

```bash
npx skills add dolo/humanizer-zh-cn --global --agent '*'
```

这会为 Skills CLI 支持的 Agent 安装 Humanizer。省略 `--global` 可只安装到当前项目。若 Agent 不受 Skills CLI 支持，复制整个仓库或至少保留 `SKILL.md` 与 `references/` 的相对目录结构。

## 使用

直接调用：

```text
/humanizer

[粘贴文本]
```

也可以写：

```text
请用 Humanizer 改写这段文字，保留事实和我的语气：[文本]
```

处理文件时给出路径：

```text
用 Humanizer 修改 docs/launch-post.md 中的正文
```

Humanizer 会先读完当前文件，只改正文。代码块、行内代码、命令、路径、YAML 元数据、数据、链接目标和引用应原样保留。

默认只返回最终文本，不附评分、草稿、分析报告或前后对比。需要分析时明确提出：

```text
请分析这段文字的 AI 写作痕迹，说明哪些地方不该改，然后给出终稿。
```

### 匹配作者风格

提供两三段作者自己的文字作为样例。Humanizer 会匹配句长、用词、标点、开头、过渡和有意的语言习惯，而不是把所有文本改成同一种“自然风格”。风格样例可以覆盖标点偏好，不能覆盖事实约束。

### 中英文混合文本

Humanizer 根据正文而不是指令语言选择规则。路由看主句语言，不看中英文字符比例：中文主句中的 `API`、`cache` 等术语不触发英文规则，独立英文句子才使用英文规则。跨段结构以主导语言判断，重复和事实校验仍作用于全文。不会把英文的连字符、标题大写或系动词规则套到中文，也不会擅自翻译其中一种语言。

## 工作流程

1. **文本分析：** 检查词汇、句式、段落、组织、重复、逻辑、节奏、具体性和对话残留。
2. **问题评估：** 区分明显问题、弱信号、正常表达、有意修辞、专业规范和作者风格。没有问题时原样返回。
3. **针对性改写：** 删除空转、包装和重复，调整机械结构与节奏，只处理实际影响阅读的部分。
4. **事实与语义校验：** 对照原文检查事实、数字、术语、引用、条件、否定、因果、观点和省略内容。
5. **最终输出：** 默认只给终稿；用户要求分析时才说明问题与取舍。

技术文档、新闻、汇报、公文、邮件、社交媒体、评论、学术文本、聊天和小说采用不同的保留标准。完整场景表位于 `SKILL.md`。

## The 50 patterns

规则全局连续编号。英文保留原来的 1–26，中文为 27–50；每种语言内部按问题强度和常见程度排列。标注“单独证据较弱”的规则必须结合其他问题，不能看到词语就改。

### English patterns（1–26）

#### A. Staging instead of stating

| # | Pattern | Before | After |
|---|---------|--------|-------|
| 1 | **Not X but Y** | “It's not just X, it's Y” | State the point directly; keep real corrections |
| 2 | **One-line closers and dramatic fragments** | “That is the real win.” after every section | Cut repetition; merge empty fragments |
| 3 | **Sayings that sound deep** | “At its core, what matters is...” | Replace empty depth with the actual claim |
| 4 | **Staged run-up before the point** | “Let's dive in”; “The hard part is...” | Remove the run-up and name the subject |
| 5 | **Arguing with no one** | “A tempting approach would be...” | Remove unraised objections; keep real alternatives |

#### B. Rhythm by rule

| # | Pattern | Before | After |
|---|---------|--------|-------|
| 6 | **Forced triads** | “innovation, inspiration, and insights” | Use only the items the meaning needs |
| 7 | **Repeated sentence openings** | “She noted... She noted...” | Merge or vary only mechanical repetition |
| 8 | **Dashes as the universal connector** (*weak alone*) | Dashes between every clause | Use clearer punctuation; match the writer's sample |
| 9 | **Stacked qualifiers** (*weak alone*) | “could potentially possibly be argued” | Keep qualifiers supported by the evidence |
| 10 | **Hyphenated pairs everywhere** (*weak alone*) | “the report is high-quality” | Apply English compound-modifier grammar |
| 11 | **Passive voice and missing subjects** (*weak alone*) | “No configuration file needed” | Name the actor only when that improves clarity |

#### C. Inflation and borrowed authority

| # | Pattern | Before | After |
|---|---------|--------|-------|
| 12 | **Overused AI words** | “delve... testament... landscape” | Use precise plain words; keep technical meanings |
| 13 | **Inflated significance** | “marking a pivotal moment” | Keep the fact; drop unsupported importance |
| 14 | **Vague connection or association** | “associated with the leadership” | Name the relationship only if known |
| 15 | **Shallow -ing phrases** | “symbolizing... reflecting...” | Keep only supported claims |
| 16 | **Sales language** | “nestled within the breathtaking region” | State concrete qualities |
| 17 | **Borrowed authority** | “Experts believe...” without a source | Use an available checked source or cut the claim |
| 18 | **Avoiding is, are, and has** | “serves as... boasts...” | Use simple verbs when they preserve the meaning |

#### D. Formatting by rule

| # | Pattern | Before | After |
|---|---------|--------|-------|
| 19 | **Bold as decoration** | Every list item has a bold label | Keep useful emphasis; remove decoration |
| 20 | **Decorative headings** | “🚀 Launch Phase” | Name the content and follow the publication style |
| 21 | **Curly quotation marks** (*weak alone*) | Quotes differ from the target format | Match the format; do not apply this to Chinese |

#### E. Leftovers from the chat and the draft

| # | Pattern | Before | After |
|---|---------|--------|-------|
| 22 | **Chatbot residue** | “Great question! ... I hope this helps!” | Keep the content; keep real message politeness |
| 23 | **Knowledge-limit disclaimers and guesses** | Missing sources followed by a plausible guess | State a relevant gap or cut the guess |
| 24 | **A heading repeated in the first sentence** | “## Performance” + “Speed matters.” | Remove only the repeated warm-up |
| 25 | **Writing about the document instead of its subject** | Narrating how the document was assembled | Describe the subject; keep useful source notes |

#### F. Writing for the wrong reader

| # | Pattern | Before | After |
|---|---------|--------|-------|
| 26 | **Re-explaining what the reader knows** | A reply repeats all context before its decision | Lead with the decision and the decisive reason |

### 中文规则（27–50）

#### G. 套话、包装与虚假深度

| # | 规则 | 识别示例 | 处理方法 |
|---|------|----------|----------|
| 27 | **空泛套话与自我预告** | “随着……不断发展”“下面将介绍” | 删除没有背景或导航价值的开场 |
| 28 | **抽象包装与名词化** | “赋能”“打造闭环”“多维度协同” | 写出原文已有的动作、对象和结果 |
| 29 | **夸大意义与强行升华** | “重要意义”“为行业树立标杆” | 保留事实和有依据的评价，删去拔高 |
| 30 | **伪深刻与空泛判断** | “从本质上来说”“是一面镜子” | 用真实机制或普通判断替代深度外壳 |
| 31 | **营销化与赞美堆叠** | “震撼、细腻、极致” | 保留参数、功能和真实感受 |
| 32 | **模糊归因与借来的权威** | “业内普遍认为”“专家表示” | 使用已有来源；不编造权威或研究 |

#### H. 机械句式与节奏

| # | 规则 | 识别示例 | 处理方法 |
|---|------|----------|----------|
| 33 | **机械否定对照** | “不是简单……而是……” | 删除无人持有的否定；保留真实纠错 |
| 34 | **固定递进与并列** | “不仅……更……”“既……又……” | 按真实关系合并或陈述各项 |
| 35 | **强行三项与数字框架** | 所有主题都凑成三个关键词 | 项目数量服从内容，不刻意换数量 |
| 36 | **伪范围与万能过渡** | 无连续范围的“从……到……” | 直接列举；保留真实跨度和顺序 |
| 37 | **重复句首与连接词堆叠** | 连续“因此”“我们需要” | 合并、省略或调整语序，保留必要衔接 |
| 38 | **自问自答与教程腔** | “为什么……？答案是……” | 直接回答；保留真实问题和人物语气 |

#### I. 重复、冗余与信息不足

| # | 规则 | 识别示例 | 处理方法 |
|---|------|----------|----------|
| 39 | **同义反复与解释复述** | “复杂且繁琐”并反复总结 | 合并同义项，删除无新信息的复述 |
| 40 | **逐段小结与结尾复读** | 每段“由此可见”，结尾再总结 | 删除复述，保留新结论和行动建议 |
| 41 | **抽象代替具体** | “采取措施并取得显著成效” | 使用原文事实；缺失时不编造 |
| 42 | **过度对冲与无关免责声明** | “某种程度上或许可能” | 保留证据需要的一层限定和真实限制 |

#### J. 模板结构与装饰

| # | 规则 | 识别示例 | 处理方法 |
|---|------|----------|----------|
| 43 | **机械分点与编号** | 短句硬拆成“一是、二是、三是” | 保留步骤和条款，其他内容按需要成段 |
| 44 | **标题与 Markdown 过度** | 短文多层标题、逐词加粗 | 保留导航价值，删除装饰和标题复述 |
| 45 | **均匀节奏与过度对仗** | 句长一致、四字格和金句堆叠 | 按语义调整，不故意制造碎句或错误 |
| 46 | **伪书面与翻译腔** | “进行一个……的动作”“被”字堆叠 | 用自然中文；保留规范语和正常主语省略 |
| 47 | **标点装饰与插入过量** | 破折号、括号和叹号密集 | 按语义整理；不改代码和中文正常标点 |

#### K. 对话残留与错误读者

| # | 规则 | 识别示例 | 处理方法 |
|---|------|----------|----------|
| 48 | **AI 对话残留** | “希望这对你有帮助”“我可以继续” | 独立成稿时删除；保留真实邮件礼貌 |
| 49 | **知识边界与无来源猜测** | 没有资料却猜测童年或动机 | 说明真实缺口或删除猜测 |
| 50 | **复述读者已知背景** | 回复重述全部背景后才给决定 | 有上下文时先答复，保留关键理由 |

完整识别标准、改写例子和“不要处理”的边界见 `references/en.md` 与 `references/zh-CN.md`。中文规则的研究依据与局限见 `docs/research-zh-CN.md`。

## 测试

包校验和运行器单测不需要第三方依赖：

```bash
python3 scripts/validate-package.py
python3 tests/test_output_checks.py
python3 tests/run_behavior_tests.py --list
```

17 个行为案例包含中文技术文章、技术文档、新闻、汇报、公文、邮件、社交媒体、评论、学术、聊天、小说、中英混合和英文回归等场景。测试案例在 `tests/cases.json`，运行器会调用真实 Agent，并检查必留事实、关键数字关系、应删表达、禁止新增数字、是否误改和输出格式。

```bash
python3 tests/run_behavior_tests.py --agent-command "claude -p --tools=\"\""
```

Agent 命令必须能从标准输入读取 UTF-8 提示，并把 UTF-8 最终文本写到标准输出。`--agent-command` 使用 POSIX quoting；Windows 的可执行文件路径应加引号，尤其是包含空格或反斜杠时。可以用任何满足这个接口的 Agent 替换示例命令。运行器不随 Skill 自动执行，也不是使用 Humanizer 的运行时依赖。自动断言不能证明任意两段文字语义等价，发布前仍需人工复核事实、归因和作者立场。

## 项目结构

- `SKILL.md`：共同五阶段流程、语言路由、输出约定和场景适配。
- `references/en.md`：英文 1–26。
- `references/zh-CN.md`：中文 27–50。
- `docs/research-zh-CN.md`：中文研究依据、分类原则和限制。
- `tests/`：行为案例与轻量运行器。
- `scripts/validate-package.py`：包结构、版本、规则编号和文档同步校验。
- `.claude-plugin/`、`.cursor-plugin/`、`agents/`、`package.json`：原有安装入口。

## 来源与许可

英文规则源自 Wikipedia 的 [Signs of AI writing](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing) 和 [WikiProject AI Cleanup](https://en.wikipedia.org/wiki/Wikipedia:WikiProject_AI_Cleanup)。原项目作者为 Siqi Chen（[@blader](https://github.com/blader)）。El Attar、Dönmez、Maurer 和 Falenska 的[跨领域英文研究](https://arxiv.org/abs/2606.04177)用于说明语言信号会随模型和文本类型变化。

中文研究参考朱君辉等人的 [《人工智能生成语言与人类语言对比研究——以 ChatGPT 为例》](https://aclanthology.org/2023.ccl-1.46/)，以及[《大模型生成回答与人类回答文本的语言特征比较研究》](http://jcip.cipsc.org.cn/cn/article/pdf/preview/zwxxxb_3707.pdf)。这些研究支持把词汇、句法、衔接和重复作为检查维度，不支持用单个特征判断某段文字的作者。

版本记录见 [CHANGELOG.md](CHANGELOG.md)。许可为 [MIT](LICENSE)。
