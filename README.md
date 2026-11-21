# ScholarScout

**一条命令看清一个研究方向在顶会上的版图：抓取 OpenReview 论文，自动分类，画出分布图和年度趋势。**

[![Python](https://img.shields.io/badge/Python-3.7+-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

<div align="center">
<img src="out/icl_pie_donut_refined.png" width="45%" alt="ICL 论文类别分布饼图">
<img src="out/icl_trend_lines_refined.png" width="45%" alt="ICL 论文各类别年度趋势图">
</div>

默认聚焦 **In-Context Learning（ICL）**，内置 9 个类别。只改配置就能换成多模态、强化学习、图神经网络等任何方向，不需要改代码。

## 为什么做 ScholarScout

写综述、找选题或者准备组会时，常常需要回答：这个方向今年在 ICLR / ICML 上有多少篇？分成哪几个子方向？哪个子方向在升温？

手动翻 OpenReview 再逐篇归类太慢。ScholarScout 把抓取、筛选、分类、统计和画图串成一条流水线，几分钟内给出带图表的答案。

## 亮点

- **自动抓取**：通过 OpenReview API 批量获取 ICLR、ICML 等会议多个年份的论文元数据。
- **三种分类方式**：规则分类（按关键词，速度快、免费）、LLM 分类（更准，按 API 计费），以及混合策略（优先用 LLM，置信度低时回退到规则）。
- **任意研究方向**：用 `--topic` 和 `--categories` 自定义主题与类别，不用改代码。
- **适合大批量处理**：LLM 结果自动缓存，可限制请求速率，支持断点续传。
- **直接出图**：生成类别分布饼图和年度趋势折线图，中文字体自动适配。
- **兼容多种模型**：支持 OpenAI、智谱 GLM，以及 vLLM / Ollama 等任何 OpenAI 兼容接口。

## 快速开始

```bash
git clone https://github.com/holamochi/scholar-scout.git
cd scholar-scout
pip install requests pandas tqdm matplotlib

python3 openreview_icl_crawl_and_plot.py --years 2024   # 规则分类，不需要 API
ls out/                                                 # CSV 和图表都在这里
```

需要 Python 3.7+，并且能访问 OpenReview API。

### 启用 LLM 分类

```bash
pip install openai
```

在仓库根目录创建 `config.json`（已列入 `.gitignore`，不会被提交）：

```json
{
  "api": {
    "api_base": "https://api.openai.com/v1",
    "api_key": "YOUR_API_KEY_HERE",
    "model": "gpt-3.5-turbo",
    "max_rpm": 60
  },
  "data_fetch": {
    "years": [2023, 2024, 2025],
    "conferences": ["ICLR", "ICML"]
  },
  "classification": {
    "use_llm": false,
    "llm_confidence_threshold": 0.6
  }
}
```

```bash
python3 openreview_icl_crawl_and_plot.py --use_llm --years 2024
```

配置优先级：命令行参数 > `config.json` > 默认值。

## 常用命令

```bash
# 指定会议和年份
python3 openreview_icl_crawl_and_plot.py --confs ICLR --years 2024 2025

# 研究其他方向：类别格式为「类别名:关键词1,关键词2;下一个类别:关键词A,关键词B」
python3 openreview_icl_crawl_and_plot.py \
  --topic "multimodal learning" \
  --categories "视觉语言融合:vision,language,VLM;跨模态检索:retrieval,cross-modal;图像生成:generation,diffusion"

# 换用智谱或本地模型
python3 openreview_icl_crawl_and_plot.py --use_llm \
  --llm_api_base "https://open.bigmodel.cn/api/paas/v4" --llm_model "glm-4"
python3 openreview_icl_crawl_and_plot.py --use_llm \
  --llm_api_base "http://localhost:8000/v1" --llm_model "llama-3-8b" --llm_api_key "dummy"

# 大批量：限速 + 断点续传（中断后重新运行同一条命令即可继续）
python3 openreview_icl_crawl_and_plot.py --use_llm --llm_max_rpm 20 --llm_batch_size 5 \
  --checkpoint_file checkpoint.json

# 只用已有 CSV 重新画图；中文显示为方框时指定字体
python3 openreview_icl_crawl_and_plot.py --plot_only --font "PingFang SC"
```

辅助脚本：`reclassify_with_llm.py`（重新分类已有数据）、`test_llm_classify.py`（测试 LLM 分类）、`check_api_models.py`（检查 API 连通性和可用模型）。

## 输出

```
out/
├── icl_papers_filtered.csv              # 简化版论文列表
├── icl_papers_classified_detailed.csv   # 详细版：含分类方法、置信度和分类理由
├── icl_pie_donut_refined.png            # 类别分布饼图
├── icl_trend_lines_refined.png          # 年度趋势图
└── llm_cache.json                       # LLM 结果缓存
```

详细版 CSV 的关键字段：`category_label`（类别）、`method`（`rule` / `llm` / `hybrid`）、`confidence`（仅 LLM）、`reasoning`（仅 LLM）。

## 内置的 ICL 分类体系

| 类别 | 研究方向 |
| --- | --- |
| Prompt 工程与优化 | 示例选择、排序、模板设计 |
| 推理与思维链 | CoT、多步推理、自洽性 |
| 机理理解与可解释性 | 机制分析、诱导头、电路分析 |
| 模型训练与架构 | 预训练方法、架构变体 |
| 效率优化 | 上下文压缩、KV 缓存、长度外推 |
| 评测基准与数据集 | 基准构建、评测方法 |
| Agent 与工具使用 | 规划、工具调用 |
| 可靠性与安全 | 校准、鲁棒性、隐私 |
| 特定技术方法 | kNN-ICL、从错误中学习 |

## 常见问题

- **图表中文显示为方框**：加 `--font "PingFang SC"`（macOS）或 `--font "Microsoft YaHei"`（Windows）。
- **LLM 调用失败**：检查 API Key、地址和模型名，可以先运行 `python3 check_api_models.py` 测试。
- **「其他 / 未归类」占比太高**：改用 `--use_llm`，或在 `--categories` 中补充关键词。
- **保护 API Key**：把密钥写在 `config.json` 里（已被 `.gitignore` 忽略），不要在命令行里直接传入。

## 代码结构

| 文件 | 职责 |
| --- | --- |
| `openreview_icl_crawl_and_plot.py` | 主程序入口 |
| `config_loader.py` | 配置管理 |
| `icl_fetcher.py` | OpenReview 数据抓取 |
| `icl_taxonomy.py` | ICL 分类体系定义 |
| `icl_classifier.py` | 分类（规则 / LLM / 混合） |
| `icl_plotter.py` | 可视化 |
| `custom_taxonomy.py` | 自定义主题扩展 |

## Topic / Keywords

- **解决的问题**：想了解某个研究方向在顶会（ICLR、ICML 等）上的论文规模、子方向构成和逐年趋势时，手动翻 OpenReview 并逐篇归类非常耗时。
- **实现方式**：Python 脚本通过 OpenReview API 抓取论文元数据，按主题关键词筛选，用「关键词规则 + LLM」混合分类，LLM 低置信度时回退到规则，并缓存结果、支持断点续传，最后输出 CSV 和 matplotlib 图表；主题和类别都可以通过配置或命令行自定义。
- **项目类别**：文献计量 · 学术论文分析 · 研究趋势可视化 · LLM 文本分类 · Python 命令行工具
- **容易混淆**：它不是论文检索或阅读工具，也不下载 PDF 全文；分析的是 OpenReview 上的标题和摘要等元数据。
- **中文关键词**：顶会论文统计、OpenReview 爬虫、ICLR 论文分析、ICML 论文分析、研究趋势、文献计量、论文分类、上下文学习、综述选题、LLM 分类
- **English keywords**: OpenReview crawler, OpenReview API, ICLR papers, ICML papers, conference paper analysis, research trend analysis, bibliometrics, paper classification, in-context learning survey, LLM classifier, hybrid rule-based classification, matplotlib, Python CLI

## TL;DR

> ScholarScout：输入一个研究方向，自动从 OpenReview 抓取 ICLR / ICML 论文，完成分类，并画出类别分布和年度趋势图。

## Link

- GitHub 仓库：https://github.com/holamochi/scholar-scout
- 同类项目：
  - Paper Copilot paperlists（整理好的顶会论文与评审数据）：https://github.com/papercopilot/paperlists
  - ICLR2020-OpenReviewData（爬取并可视化 ICLR 论文与评审）：https://github.com/shaohua0116/ICLR2020-OpenReviewData
  - openreview-visualization（ICLR 投稿数据可视化）：https://github.com/ranpox/openreview-visualization

## 开源许可

[MIT](LICENSE)。感谢 [OpenReview](https://openreview.net/) 提供的开放 API。
