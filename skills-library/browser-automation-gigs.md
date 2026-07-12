# 🖥️ 浏览器自动化接单完全指南

> 用 Playwright/Puppeteer 帮人做自动化任务，月入$500-2000

## 什么是浏览器自动化接单

帮企业或个人完成重复性的浏览器操作任务。例如：
- 每天自动抓取竞品价格
- 自动填写表单并提交
- 批量注册账号
- 自动发帖/留言
- 数据采集+整理成Excel

**核心优势**：不需要自己开发产品，只需要用现成工具完成客户需求。

---

## 入门准备

### 硬件要求
- 任何能上网的电脑即可
- 8GB RAM 够用

### 软件要求
- Python 3.11（本机已有）
- Playwright（已安装）
- 无头浏览器（Chrome/Firefox）

### 安装（5分钟）
```bash
~/.hermes/hermes-agent/venv/bin/pip install playwright
playwright install chromium  # 安装浏览器
```

---

## 热门接单类型

### 1. 数据采集（最常见）
**客单价**：$50-500
**典型需求**：
- "爬取某网站所有产品名称和价格"
- "采集某论坛所有用户帖子"
- "抓取竞品官网所有产品信息"

**执行示例**：
```python
from playwright.sync_api import sync_playwright
import csv

def scrape_products(url, output_file):
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        page.goto(url)
        
        products = []
        while True:
            items = page.query_selector_all('.product-item')
            for item in items:
                name = item.query_selector('.name').inner_text()
                price = item.query_selector('.price').inner_text()
                products.append({"name": name, "price": price})
            
            # 翻页
            next_btn = page.query_selector('.pagination .next')
            if not next_btn or not next_btn.is_enabled():
                break
            next_btn.click()
            page.wait_for_timeout(2000)
        
        browser.close()
        
        # 保存CSV
        with open(output_file, 'w', newline='') as f:
            writer = csv.DictWriter(f, fieldnames=["name", "price"])
            writer.writeheader()
            writer.writerows(products)
        
        return len(products)

# 使用
count = scrape_products("https://example.com/products", "products.csv")
print(f"采集了 {count} 个产品")
```

### 2. 表单自动填写
**客单价**：$30-200
**典型需求**：
- "批量在Excel里填好的数据提交到表单"
- "自动注册100个账号"

**注意事项**：
- 需要客户提供数据文件
- 注意网站的防重复提交限制
- 适当加延迟（每条间隔2-5秒）

### 3. 价格监控
**客单价**：$100-500（按月订阅$50-200/月）
**典型需求**：
- "每天监控竞品价格，发邮件通知"
- "某商品降价超过10%时提醒"

```python
from playwright.sync_api import sync_playwright
import smtplib
from email.mime.text import MIMEText
from datetime import datetime

def monitor_price(url, target_price, email_to):
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=True)
        page = browser.new_page()
        page.goto(url)
        page.wait_for_selector('.price')
        
        price_text = page.query_selector('.price').inner_text()
        # 提取数字
        price = float(''.join(filter(str.isdigit, price_text.split('-')[0])))
        
        browser.close()
        
        if price <= target_price:
            msg = MIMEText(f"{url} 的价格现在是 ${price}，低于目标价 ${target_price}")
            msg['Subject'] = '价格提醒'
            msg['To'] = email_to
            # 发送邮件（需配置SMTP）
            print(f"⚠️  价格触发: ${price}")
            return True
    
    return False
```

### 4. 社交媒体自动化
**客单价**：$50-300
**典型需求**：
- "自动在多个平台发布相同内容"
- "批量给用户发私信"

**警告**：遵守各平台规则，高风险

---

## 接单平台

### 1. 程序员客栈（推荐国内）
https://www.proginn.com
- 任务多，结算快
- 适合中文客户

### 2. 电鸭社区
https://eleduck.com
- 远程工作为主
- 质量较高

### 3. Upwork（英文，高单价）
https://upwork.com
- 单笔$100-2000
- 需要英语沟通
- 适合技术强的

### 4. Fiverr（标准化产品）
https://fiverr.com
- 把服务打包成$5-50的产品
- 被动收入潜力大

### 5. 淘宝/闲鱼（本地化）
- 竞争激烈但需求大
- 低价走量

---

## 定价策略

| 服务类型 | 一次性 | 月订阅 |
|---------|-------|-------|
| 简单爬虫（<100条） | $30-100 | - |
| 中等爬虫（100-1000条） | $100-300 | $50-100/月 |
| 复杂爬虫（1000+条） | $300-1000 | $100-300/月 |
| 价格监控 | - | $50-200/月 |
| 表单自动化 | $30-150 | - |

**定价技巧**：
- 首次合作打9折
- 月订阅比一次性贵30%要有保障
- 明确交付范围，按范围外另收费

---

## 提案模板

发送给客户的提案：

```
主题：关于[需求描述]的解决方案

你好，

我是自动化开发工程师，熟悉 Playwright/Selenium 技术栈，
可以帮你完成[具体任务]。

方案：
1. [具体步骤]
2. [预计时间]
3. [交付物]

报价：$XXX（一次性）/ $XX/月（订阅）

优势：
- 成功率>99%
- 可定时运行
- 数据格式灵活（CSV/JSON/Excel）

请问这个方案合适吗？需要调整可以告诉我。

谢谢，
[你的名字]
```

---

## 避坑指南

### ✅ 应该做的
1. **先小后大**：第一单做简单任务，建立信任
2. **书面确认**：需求、用时、价格全部文字确认
3. **分阶段交付**：先做10%预览，满意再继续
4. **保留证据**：截图/日志记录完成的工作

### ❌ 不要做的
1. **不签合同不做**：口头承诺不可靠
2. **不收预付不做**：至少30%
3. **不做违法任务**：爬虫要遵守robots.txt，不做账号批量注册
4. **不超范围**：合同外的需求另收费

### 常见问题
**Q: 客户跑单怎么办？**
A: 预付款30-50%，交付前不交完整代码

**Q: 技术实现不了怎么办？**
A: 提前评估，明确告知风险，不要硬接

**Q: 怎么收款？**
A: PayPal/支付宝/微信/Payoneer，中小客户用PayPal，大客户用Payoneer

---

## 变现时间线

| 时间 | 目标 | 行动 |
|------|------|------|
| 第1周 | 接到第1单 | 注册所有平台 + 投5份标 |
| 第2周 | 完成3单 | 优先做便宜的快速完成 |
| 第1月 | $200 | 口碑积累 + 提价 |
| 第3月 | $500-1000 | 月订阅客户稳定 |
| 第6月 | $2000+ | 建立稳定的客户群 |

---

## 快速起步清单

- [ ] 安装 Playwright：`pip install playwright && playwright install chromium`
- [ ] 在程序员客栈完善个人主页
- [ ] 在 Upwork 创建 Profile
- [ ] 制作一个演示爬虫（抓任意网站）
- [ ] 投递第一个任务
- [ ] 设置 PayPal 收款

---

*你已有工具：~/Projects/browser-agent/ — 基础框架已就绪，可直接改造使用*