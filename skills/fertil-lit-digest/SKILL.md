---
name: fertil-lit-digest
description: 生殖医学领域（男性不育/精子质量、卵巢发育/衰老/生殖疾病等）文献的公众号编译流程：PMID 标题与摘要翻译（三段式+编译性导读）、约200字本期看点导读、其他文献列表两行式格式化
type: prompt
whenToUse: 用户要求翻译 PMID 文献的标题和摘要、为"精子质量及男性不育的分子遗传学新发现"或"卵巢发育、衰老及相关疾病的遗传新发现"等系列公众号推文撰写导读，或格式化文献列表时
---

# 生殖医学文献公众号编译流程

用于生殖医学领域系列公众号推文（如"精子质量及男性不育的分子遗传学新发现"、"卵巢发育、衰老及相关疾病的遗传新发现"）。按用户当前需求执行以下一个或多个阶段。不同系列仅导读模板和术语表侧重不同，流程通用。

## 阶段一：标题与摘要翻译

### 1. 获取原文

按优先级获取标题与摘要（用 Bash curl，注意网络编码问题）：

1. Europe PMC REST（本环境实测稳定可用；含结构化摘要标签，需剥 HTML；摘要可能截断，注意用字段内偏移补全）：
   `curl -s "https://www.ebi.ac.uk/europepmc/webservices/rest/search?query=EXT_ID:<PMID>&format=json&resultType=core"`
2. NCBI E-utilities（纯文本最干净）：
   `curl -s "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id=<PMID>&rettype=abstract&retmode=text"`
3. 备用：Crossref API、Semantic Scholar API（`api.semanticscholar.org/graph/v1/paper/DOI:<doi>?fields=title,abstract`）。

注意：PubMed 网页端有反爬（PoW challenge/cookies 验证），FetchURL 直接抓取常失败，优先走上述 API；Google/ScienceDirect 常有 reCAPTCHA/403。部分短通讯各数据库均无摘要——此时如实告知用户"无正式摘要"，可据全文或团队官方新闻稿编译"准摘要"，并明确标注来源性质。若用户提供本地 PDF/MD 全文，用 `pdftotext -enc UTF-8`（Git Bash 自带）或直接 Read 提取。

### 2. 翻译规则

- 先给**标题译文**（加粗），再给摘要译文；注明期刊、年份、DOI 及来源链接（Europe PMC）。
- 摘要按"背景—方法与结果—结论"拆为**三段式**，即使原文是单段非结构式摘要。
- 默认字数约 400–550 字：研究背景段从简，中间段以实验结果、观点、机制等关键信息为主，结论段精简；用户另行指定字数时从其要求。
- **编译性导读**：仅当摘要本身不含背景句时，在段首插入一句编译性导读，并以"（此句为编译性导读，非摘要原文）"标注；摘要首句已是背景则不加。
- **忠实原文**：不添加原文没有的修饰（如原文无 significantly 就不写"显著"）；"highlight"译"凸显"而非"揭示"；"anti-X therapy"译"抗 X 治疗"；elevation 译"升高"而非"上调"；cell viability 译"细胞活力"而非"细胞存活率"。
- 保留关键量化数据（样本量、剂量、时间点、P 值、95%CI、SMD、I² 等）。
- 用户要求控制字数时，优先保留核心结果数据，删机制细节与次要内容；"机制上"等衔接语非必须，可按需省略。
- 如从全文补充中段实验结果，按原文 Results 顺序整合，数据点须出自原文；之后在"编译说明"中注明补充来源。
- 文末附简短**编译说明/微调说明**：段落拆分依据、术语选择、相对原文的增删。用户给出自己修改的版本时，在其基础上微调并逐条说明改动理由，不重写。

### 3. 术语规范（固定译名）

| 英文 | 中文 |
|---|---|
| premature ovarian insufficiency (POI) | 早发性卵巢功能不全（**勿漏"早发性"**） |
| premature ovarian failure (POF) | 卵巢早衰 |
| diminished ovarian reserve (DOR) | 卵巢储备功能减退 |
| polycystic ovary syndrome (PCOS) | 多囊卵巢综合征 |
| granulosa cells (GCs) | 颗粒细胞 |
| neddylation | 拟素化修饰 |
| zygotic genome activation (ZGA) | 合子基因组激活 |
| poor ovarian responder | 卵巢低反应（患者） |
| Sertoli cells | （睾丸）支持细胞 |
| blood-testis barrier | 血睾屏障 |
| orchitis / epididymitis | 睾丸炎 / 附睾炎 |
| asthenozoospermia / oligozoospermia / teratozoospermia | 弱精子症 / 少精子症 / 畸形精子症 |
| oligoasthenoteratozoospermia (OAT) | 少弱畸形精子症 |
| autophagic flux | 自噬通量 |
| tight junction | 紧密连接 |
| sperm capacitation / acrosome reaction | 精子获能 / 顶体反应 |
| mini-review | 微综述（**勿译"小综述"**） |
| meta-synthesis / meta-analysis | 荟萃合成 / 荟萃分析（注意区分） |
| in silico | 生物信息学（分析） |
| serotonin | 5-羟色胺（血清素） |

基因名、蛋白名保持原文（如 Il11ra1、ULK1、MXRA7、CFAP44）；缩写首次出现给中文全称。领域专有名词无通行中译者**保留原文不译**（如 manchette、CatSper）。

## 阶段二：本期看点导读

- 约 200 字。按系列选用模板：

**男性不育系列**：

> 本期看点继续收集与精子质量和男性生殖健康有关的前沿进展。涉及……。多项研究……。……（靶点/标志物亮点收尾）。

**卵巢系列**：

> 本期看点继续收集近期与卵巢发育、生殖寿命相关的分子和遗传学前沿研究。涉及……（疾病机制）。多项研究……（方法/技术）。……（治疗转化/其他亮点）。

- 概括本期所有已译文文献，按"机制 → 方法/技术 → 标志物/治疗转化"归类点题，列举代表性分子/通路/药物；形式不必与模板完全一样。

## 阶段三：其他文献列表格式化

输入通常是混排文本（标题、作者、期刊、PMID、机翻译文交错，格式可能不统一）。对每条输出两行：

```
Original English Title.
中文标题译文。*PubMed期刊缩写*


```

- 上行：仅原始英文标题，以 `.` 结尾。
- 下行：**重新翻译**（不在原有机翻基础上修改），以 `。` 结尾，紧跟斜体期刊名缩写（Markdown `*...*`），用 PubMed 标准缩写（如 *Hum Reprod*、*Proc Natl Acad Sci U S A*、*Asian J Androl*）；不确定时用 NCBI 查准。
- 每条之间空两行。
- **同一期刊的条目相邻排列**（按期刊分组微调顺序）。
- 清理：删除"（精子质量）"等内部分类标记；修正明显机翻错误（如"Guilu Erxian"应为"龟鹿二仙"而非"桂露二仙"、标题重复粘连）；专有名词保留原文。
- 条目较多时写入 Markdown 文件交付，并在回复中说明所做修正。

## 通用要求

- 全程用中文与用户交流；术语、基因名、期刊名保留原文。
- 译文写文件前无需征得同意；涉及 git 操作需确认。
- 拿不准的译法或原文无依据的内容，明确向用户指出而非掩饰。
- 交付给微信公众号后台的文本若含斜体（如期刊名），提醒用户：编辑器内可见的粘贴斜体发布后可能被清洗丢失，需用编辑器自带斜体按钮重设，或改用 mdnice 等工具转换。
