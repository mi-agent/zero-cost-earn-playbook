# ⚡ 快速变现清单 - 本周就能做

> 按投入产出比排序，越靠前越快见钱

---

## 🔥 本周立即行动（按优先级）

### 1. Gumroad 上架数字产品（30分钟）
**收入潜力**：$29 × 5单 = $145
**操作**：
```bash
# 你已有的产品
ls ~/Projects/PrivacyAI-Business-Bundle.zip

# 上 Gumroad
# 1. 打开 gumroad.com 用 Google 登录
# 2. 点击 New Product
# 3. 上传 zip 文件
# 4. 标题：PrivacyAI Business Bundle
# 5. 定价：$29
# 6. 发布
```
**立刻做**：今天就上架

---

### 2. 发布今日内容到知乎/掘金（20分钟）
**收入潜力**：被动流量 → 联盟点击
**操作**：
```bash
# 查看生成的内容
cat ~/Projects/task-bot/content/latest_zhihu.md | head -40
```
**立刻做**：复制发布到知乎，今天发一篇

---

### 3. 投递3个任务申请（30分钟）
**收入潜力**：$50-500/单
**操作**：
```bash
# 查看高薪任务
~/.hermes/hermes-agent/venv/bin/python -c "
import json
with open('/Users/ai/Projects/task-bot/data/latest_final.json') as f:
    d = json.load(f)
for j in d.get('jobs',[])[:10]:
    print(f'  {j[\"title\"][:50]}')
    print(f'  {j[\"url\"]}')
    print()
"
```
**立刻做**：选3个，投递

---

### 4. 申请2个联盟项目（15分钟）
**收入潜力**：被动收入
**操作**：
- PIA VPN 联盟：https://www.privateinternetaccess.com/affiliates
- DigitalOcean 推荐：https://www.digitalocean.com
- ShareASale：https://www.shareasale.com

**立刻做**：今天申请

---

### 5. 配置 Telegram Bot（10分钟）
**收入潜力**：每日任务推送 → 不错过好任务
**操作**：
```bash
~/.hermes/hermes-agent/venv/bin/python ~/Projects/task-bot/notifiers/notifier.py --setup
```
**立刻做**：配置好，每天自动收到任务

---

## 📋 今日任务清单（2小时完成）

```
□ 1. Gumroad 上架数字产品（30分钟）
□ 2. 发一篇知乎文章（20分钟）
□ 3. 投递3个任务申请（30分钟）
□ 4. 申请2个联盟（15分钟）
□ 5. 配置 Telegram（10分钟）
□ 6. 复盘并记录（5分钟）
```

---

## 💰 本周收入目标

| 来源 | 目标 | 具体行动 |
|------|------|---------|
| 数字产品 | $50 | 上架 + 分享 |
| 接单 | $100 | 3个申请 |
| 联盟 | $10 | 10个点击 |

**本周目标：$160**

---

## ⏰ 每日小任务（10分钟）

每天花10分钟做一件事：

| 日期 | 任务 |
|------|------|
| 今天 | 上 Gumroad |
| 明天 | 发知乎 |
| 后天 | 投递任务 |
| 第4天 | 申请联盟 |
| 第5天 | 配置推送 |
| 第6天 | 复盘调整 |
| 第7天 | 庆祝第一笔收入！ |

---

## 🎯 里程碑

- [ ] **Day 1**：Gumroad 上架完成
- [ ] **Day 2**：第一篇知乎发布
- [ ] **Day 3**：第一个任务申请
- [ ] **Day 7**：第一笔收入
- [ ] **Day 30**：月入 $100

---

## 🚀 下周扩展

一旦本周基础打好，下周开始：
- 把内容发到更多平台（小红书、掘金）
- 提高任务申请频率
- 开始收集邮件订阅
- 申请更多联盟项目

---

*不要想太多，先做再优化。*

*第一个$100比完美的系统更重要。*