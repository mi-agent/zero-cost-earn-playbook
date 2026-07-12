# 🚀 零成本变现 - 速查卡

## ⚡ 今日必做（30分钟出成果）

### 🔥 第一件事：发布内容

```bash
# 查看今日生成的内容素材
cat ~/Projects/task-bot/content/latest_zhihu.md | head -30

# 或者查看任务列表
cat ~/Projects/task-bot/data/latest_final.json | python3 -c "import json,sys; d=json.load(sys.stdin); [print(f\"  • {j['title'][:50]} | {j['salary']}\") for j in d.get('jobs',[])[:5]]"
```

**行动**：复制内容 → 发布到知乎/掘金

---

### 💰 第二件事：接单

```bash
# 查看高薪任务（15k+）
~/.hermes/hermes-agent/venv/bin/python -c "
import json
with open('/Users/ai/Projects/task-bot/data/latest_final.json') as f:
    d = json.load(f)
for j in d.get('jobs',[]):
    s = j.get('salary','')
    if '15' in s or '20' in s or '25' in s or '30' in s or '50' in s:
        print(f'  {j[\"title\"][:50]} | {s}')
"
```

**行动**：打开最高薪的3个任务 → 用提案模板投递

---

## 📋 技能速查

| 技能 | 你现在需要做什么 | 产出目标 |
|------|-----------------|---------|
| **内容营销** | 复制 latest_zhihu.md 发知乎 | 每周5篇 |
| **接单** | 挑15k+任务投递 | 每周3份申请 |
| **联盟营销** | 注册ShareASale + 嵌入链接 | 5个项目 |
| **数字产品** | Gumroad上架zip包 | 1个产品 |

---

## 🔗 关键链接

| 平台 | 用途 | 链接 |
|------|------|------|
| GitHub Pages | 内容站 | https://mi-agent.github.io/ai-tools-guide/ |
| Gumroad | 数字产品 | https://gumroad.com（登录） |
| ShareASale | 联盟平台 | https://www.shareasale.com |
| 远程.work | 任务来源 | https://yuancheng.work |
| 电鸭社区 | 任务来源 | https://eleduck.com |
| 程序员客栈 | 任务来源 | https://www.proginn.com |

---

## 📁 关键文件

```
任务数据:    ~/Projects/task-bot/data/latest_final.json
内容素材:    ~/Projects/task-bot/content/latest_*.md
提案模板:    ~/Projects/evolution-engine/feedback/proposal_template.md
学习追踪:    ~/Projects/learning-tracker/learning_tracker.py
数字产品:    ~/Projects/PrivacyAI-Business-Bundle.zip
自动化:      ~/Projects/task-bot/master_controller.py
进化引擎:    ~/Projects/evolution-engine/evolution.py
```

---

## ⚡ 命令速查

```bash
# 查看当前状态
~/.hermes/hermes-agent/venv/bin/python ~/Projects/learning-tracker/learning_tracker.py status

# 运行完整自动化
bash ~/Projects/task-bot/run.sh

# 深度进化分析
~/.hermes/hermes-agent/venv/bin/python ~/Projects/evolution-engine/evolution.py

# 添加收入
~/.hermes/hermes-agent/venv/bin/python ~/Projects/learning-tracker/learning_tracker.py income freelance 100

# 更新技能进度
~/.hermes/hermes-agent/venv/bin/python ~/Projects/learning-tracker/learning_tracker.py skill content_marketing 50

# 生成周报
~/.hermes/hermes-agent/venv/bin/python ~/Projects/learning-tracker/learning_tracker.py report
```

---

## 🎯 今日目标

- [ ] 发布1篇知乎/掘金文章
- [ ] 投递2份任务申请
- [ ] 申请1个联盟项目（ShareASale）
- [ ] 配置好 Telegram Bot

---

*最后更新: 2026-07-12*