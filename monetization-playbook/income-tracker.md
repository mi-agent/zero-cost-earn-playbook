# 收入追踪系统

> 包含Python脚本 + Markdown模板，追踪所有收入来源

---

## 第一部分：Python追踪脚本

### 安装依赖

```bash
pip install pandas openpyxl gspread google-auth
```

### income_tracker.py

```python
#!/usr/bin/env python3
"""
收入追踪脚本
用法: python income_tracker.py
"""

import csv
import json
from datetime import datetime, timedelta
from pathlib import Path
from typing import Dict, List

# ============ 配置 ============
DATA_DIR = Path("data")
INCOME_FILE = DATA_DIR / "income.csv"
CONFIG_FILE = DATA_DIR / "config.json"

# ============ 收入类型定义 ============
INCOME_TYPES = {
    "freelance": "接单收入",      # Upwork/Fiverr/直接客户
    "affiliate": "联盟收入",      # 推广佣金
    "digital": "数字产品",        # 模板/课程/电子书
    "passive": "被动收入",        # 其他被动来源
}

PLATFORMS = {
    "upwork": "Upwork",
    "fiverr": "Fiverr",
    "direct": "直接客户",
    "amazon": "Amazon联盟",
    "clickbank": "ClickBank",
    "gumroad": "Gumroad",
    "medium": "Medium Partner",
    "other": "其他",
}

# ============ 初始化 ============
def init_data_files():
    """初始化数据文件"""
    DATA_DIR.mkdir(exist_ok=True)
    
    if not INCOME_FILE.exists():
        with open(INCOME_FILE, 'w', newline='', encoding='utf-8') as f:
            writer = csv.writer(f)
            writer.writerow([
                '日期', '类型', '平台', '项目名称', 
                '金额(USD)', '金额(CNY)', '备注', '状态'
            ])
    
    if not CONFIG_FILE.exists():
        config = {
            "exchange_rate": 7.2,
            "currency": "USD",
            "monthly_goal": 500,
            "week_goal": 125
        }
        with open(CONFIG_FILE, 'w', encoding='utf-8') as f:
            json.dump(config, f, ensure_ascii=False, indent=2)

# ============ 收入记录 ============
def add_income(
    date: str,
    income_type: str,
    platform: str,
    project_name: str,
    amount_usd: float,
    note: str = "",
    status: str = "confirmed"
) -> bool:
    """
    添加收入记录
    
    Args:
        date: 日期 YYYY-MM-DD
        income_type: freelance/affiliate/digital/passive
        platform: 平台名
        project_name: 项目名
        amount_usd: 金额（美元）
        note: 备注
        status: pending/confirmed/paid
    """
    config = load_config()
    
    with open(INCOME_FILE, 'a', newline='', encoding='utf-8') as f:
        writer = csv.writer(f)
        writer.writerow([
            date,
            income_type,
            platform,
            project_name,
            f"{amount_usd:.2f}",
            f"{amount_usd * config['exchange_rate']:.2f}",
            note,
            status
        ])
    
    print(f"✅ 记录已添加: {project_name} = ${amount_usd}")
    return True

def load_config() -> dict:
    """加载配置"""
    with open(CONFIG_FILE, 'r', encoding='utf-8') as f:
        return json.load(f)

# ============ 数据查询 ============
def get_today_income() -> Dict:
    """获取今日收入"""
    today = datetime.now().strftime('%Y-%m-%d')
    return get_income_by_date(today)

def get_week_income() -> Dict:
    """获取本周收入"""
    today = datetime.now()
    start_of_week = (today - timedelta(days=today.weekday())).strftime('%Y-%m-%d')
    return get_income_by_date_range(start_of_week, today.strftime('%Y-%m-%d'))

def get_month_income() -> Dict:
    """获取本月收入"""
    today = datetime.now()
    start_of_month = today.replace(day=1).strftime('%Y-%m-%d')
    return get_income_by_date_range(start_of_month, today.strftime('%Y-%m-%d'))

def get_income_by_date(date: str) -> Dict:
    """按日期查询"""
    return get_income_by_date_range(date, date)

def get_income_by_date_range(start_date: str, end_date: str) -> Dict:
    """按日期范围查询"""
    rows = []
    with open(INCOME_FILE, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)
        for row in reader:
            if start_date <= row['日期'] <= end_date:
                rows.append(row)
    
    total_usd = sum(float(r['金额(USD)']) for r in rows)
    config = load_config()
    
    return {
        'records': rows,
        'count': len(rows),
        'total_usd': total_usd,
        'total_cny': total_usd * config['exchange_rate']
    }

def get_platform_breakdown(start_date: str, end_date: str) -> Dict:
    """各平台收入明细"""
    data = get_income_by_date_range(start_date, end_date)
    breakdown = {p: 0 for p in PLATFORMS.values()}
    
    for row in data['records']:
        platform = row['平台']
        amount = float(row['金额(USD)'])
        breakdown[platform] = breakdown.get(platform, 0) + amount
    
    return {k: v for k, v in breakdown.items() if v > 0}

def get_type_breakdown(start_date: str, end_date: str) -> Dict:
    """收入类型分布"""
    data = get_income_by_date_range(start_date, end_date)
    breakdown = {t: 0 for t in INCOME_TYPES.values()}
    
    for row in data['records']:
        income_type = row['类型']
        amount = float(row['金额(USD)'])
        breakdown[income_type] = breakdown.get(income_type, 0) + amount
    
    return {k: v for k, v in breakdown.items() if v > 0}

# ============ 报表生成 ============
def print_daily_report():
    """打印日报"""
    today = datetime.now().strftime('%Y-%m-%d')
    data = get_income_by_date(today)
    config = load_config()
    
    print(f"\n{'='*50}")
    print(f"📊 {today} 收入日报")
    print(f"{'='*50}")
    print(f"总收入: ${data['total_usd']:.2f} (¥{data['total_cny']:.2f})")
    print(f"记录数: {data['count']}")
    
    if data['records']:
        print(f"\n明细:")
        for r in data['records']:
            print(f"  - [{r['平台']}] {r['项目名称']}: ${r['金额(USD)']}")
    
    print(f"\n周进度: ${get_week_income()['total_usd']:.2f} / ${config['week_goal']}")
    print(f"月进度: ${get_month_income()['total_usd']:.2f} / ${config['monthly_goal']}")
    print(f"{'='*50}\n")

def generate_weekly_report() -> str:
    """生成周报Markdown"""
    data = get_week_income()
    config = load_config()
    
    today = datetime.now()
    week_start = (today - timedelta(days=today.weekday())).strftime('%Y-%m-%d')
    week_end = today.strftime('%Y-%m-%d')
    
    platform_breakdown = get_platform_breakdown(week_start, week_end)
    type_breakdown = get_type_breakdown(week_start, week_end)
    
    report = f"""# 📊 周报：{week_start} 至 {week_end}

## 收入概览

| 指标 | 本周 | 周目标 | 完成率 |
|------|------|--------|--------|
| 总收入 | ${data['total_usd']:.2f} | ${config['week_goal']} | {data['total_usd']/config['week_goal']*100:.1f}% |
| 交易数 | {data['count']} | - | - |

## 收入来源

### 按平台

| 平台 | 金额 | 占比 |
|------|------|------|
"""
    for platform, amount in sorted(platform_breakdown.items(), key=lambda x: -x[1]):
        pct = amount / data['total_usd'] * 100 if data['total_usd'] > 0 else 0
        report += f"| {platform} | ${amount:.2f} | {pct:.1f}% |\n"
    
    report += f"""
### 按类型

| 类型 | 金额 | 占比 |
|------|------|------|
"""
    for income_type, amount in sorted(type_breakdown.items(), key=lambda x: -x[1]):
        pct = amount / data['total_usd'] * 100 if data['total_usd'] > 0 else 0
        report += f"| {income_type} | ${amount:.2f} | {pct:.1f}% |\n"
    
    report += f"""
## 交易明细

| 日期 | 项目 | 平台 | 类型 | 金额 |
|------|------|------|------|------|
"""
    for r in data['records']:
        report += f"| {r['日期']} | {r['项目名称']} | {r['平台']} | {r['类型']} | ${r['金额(USD)']} |\n"
    
    # 月度累计
    month_data = get_month_income()
    report += f"""
## 月度累计

| 指标 | 金额 |
|------|------|
| 本月总收入 | ${month_data['total_usd']:.2f} |
| 月目标 | ${config['monthly_goal']} |
| 完成率 | {month_data['total_usd']/config['monthly_goal']*100:.1f}% |

---
*报表生成时间: {datetime.now().strftime('%Y-%m-%d %H:%M')}*
"""
    
    return report

def save_weekly_report():
    """保存周报"""
    report = generate_weekly_report()
    today = datetime.now()
    filename = DATA_DIR / f"weekly-report-{today.strftime('%Y-W%W')}.md"
    
    with open(filename, 'w', encoding='utf-8') as f:
        f.write(report)
    
    print(f"📄 周报已保存: {filename}")
    return filename

# ============ CLI 界面 ============
def main():
    init_data_files()
    
    import argparse
    parser = argparse.ArgumentParser(description='收入追踪工具')
    parser.add_argument('action', choices=[
        'add', 'today', 'week', 'month', 'report', 'stats'
    ], help='操作类型')
    parser.add_argument('--type', '-t', choices=INCOME_TYPES.keys())
    parser.add_argument('--platform', '-p', choices=PLATFORMS.keys())
    parser.add_argument('--project', '-n', help='项目名称')
    parser.add_argument('--amount', '-a', type=float, help='金额(USD)')
    parser.add_argument('--date', '-d', help='日期 YYYY-MM-DD')
    parser.add_argument('--note', help='备注')
    
    args = parser.parse_args()
    
    if args.action == 'add':
        add_income(
            date=args.date or datetime.now().strftime('%Y-%m-%d'),
            income_type=args.type or 'freelance',
            platform=args.platform or 'other',
            project_name=args.project or '未命名项目',
            amount_usd=args.amount or 0,
            note=args.note or ''
        )
    
    elif args.action == 'today':
        print_daily_report()
    
    elif args.action == 'week':
        data = get_week_income()
        print(f"\n📊 本周收入: ${data['total_usd']:.2f}")
        print(f"记录数: {data['count']}")
    
    elif args.action == 'month':
        data = get_month_income()
        print(f"\n📊 本月收入: ${data['total_usd']:.2f}")
        print(f"记录数: {data['count']}")
    
    elif args.action == 'report':
        save_weekly_report()
        print(generate_weekly_report())
    
    elif args.action == 'stats':
        # 基础统计
        today = datetime.now()
        month_start = today.replace(day=1).strftime('%Y-%m-%d')
        
        print("\n📈 收入统计")
        print(f"月收入: ${get_month_income()['total_usd']:.2f}")
        print(f"周收入: ${get_week_income()['total_usd']:.2f}")
        print(f"今日收入: ${get_today_income()['total_usd']:.2f}")
        
        print("\n平台分布:")
        for p, a in get_platform_breakdown(month_start, today.strftime('%Y-%m-%d')).items():
            print(f"  {p}: ${a:.2f}")
        
        print("\n类型分布:")
        for t, a in get_type_breakdown(month_start, today.strftime('%Y-%m-%d')).items():
            print(f"  {t}: ${a:.2f}")

if __name__ == '__main__':
    main()
```

### 快速使用命令

```bash
# 添加收入
python income_tracker.py add -t freelance -p upwork -n "数据爬虫项目" -a 50 -d 2025-01-15

# 查看今日
python income_tracker.py today

# 查看本周
python income_tracker.py week

# 查看本月
python income_tracker.py month

# 生成周报
python income_tracker.py report

# 查看统计
python income_tracker.py stats
```

---

## 第二部分：Markdown模板

### daily-report-template.md

```markdown
# 📅 每日收入报告

**日期**: YYYY-MM-DD

## 今日收入

| 项目 | 平台 | 类型 | 金额 |
|------|------|------|------|
| | | | |

**今日合计**: $__ | ¥__

## 任务进度

### 开发中项目
- [ ] 项目1 - __%
- [ ] 项目2 - __%

### 待跟进
- [ ] Upwork提案跟进
- [ ] Fiverr客户沟通
- [ ] 账单提交

## 明日计划

- [ ] 
- [ ] 
- [ ] 

## 备注

```
自由记录
```
```

### weekly-report-template.md

```markdown
# 📊 周报：第__周 (YYYY/MM/DD - YYYY/MM/DD)

## 收入概览

| 指标 | 本周 | 较上周 | 目标 |
|------|------|--------|------|
| 总收入(USD) | $__ | __% | $125 |
| 交易数 | __ | __ | - |
| 平均客单价 | $__ | __% | - |

## 收入明细

### 按平台

| 平台 | 本周 | 占比 | 趋势 |
|------|------|------|------|
| Upwork | $__ | __% | ↑/↓/→ |
| Fiverr | $__ | __% | ↑/↓/→ |
| 联盟 | $__ | __% | ↑/↓/→ |
| 数字产品 | $__ | __% | ↑/↓/→ |
| 其他 | $__ | __% | ↑/↓/→ |

### 按类型

| 类型 | 本周 | 占比 |
|------|------|------|
| 接单收入 | $__ | __% |
| 联盟收入 | $__ | __% |
| 数字产品 | $__ | __% |
| 被动收入 | $__ | __% |

## 交易记录

| 日期 | 项目 | 客户 | 平台 | 金额 |
|------|------|------|------|------|
| | | | | |
| | | | | |
| | | | | |

## 本周成就

- [ ] 成就1
- [ ] 成就2

## 问题与解决

| 问题 | 解决方案 | 状态 |
|------|----------|------|
| | | 已解决/进行中 |
| | | |

## 下周计划

### 收入目标
- 周目标: $__
- 日均需: $__

### 重点任务
- [ ] 任务1
- [ ] 任务2
- [ ] 任务3

### 渠道优化
- [ ] 优化点1
- [ ] 优化点2

## 月度累计

| 月份 | 目标 | 实际 | 完成率 |
|------|------|------|--------|
| __月 | $500 | $__ | __% |
```

### monthly-review-template.md

```markdown
# 📆 月度复盘：YYYY年MM月

## 收入总览

| 指标 | 数值 | 环比 | 同比 |
|------|------|------|------|
| 月收入 | $__ | __% | __% |
| 工作天数 | __ | - | - |
| 日均收入 | $__ | __% | __% |
| 平均客单价 | $__ | __% | __% |
| 客户数 | __ | __ | __ |

## 收入趋势

```chart
周  周收入($)
1   __
2   __
3   __
4   __
```

## 各渠道表现

| 渠道 | 收入 | 占比 | 变化 | 评价 |
|------|------|------|------|------|
| Upwork | $__ | __% | __% | 好/中/差 |
| Fiverr | $__ | __% | __% | 好/中/差 |
| 直接客户 | $__ | __% | __% | 好/中/差 |
| 联盟 | $__ | __% | __% | 好/中/差 |
| 数字产品 | $__ | __% | __% | 好/中/差 |

## Top 5 项目

| 排名 | 项目 | 客户 | 渠道 | 金额 |
|------|------|------|------|------|
| 1 | | | | $__ |
| 2 | | | | $__ |
| 3 | | | | $__ |
| 4 | | | | $__ |
| 5 | | | | $__ |

## 里程碑

- [ ] 里程碑1
- [ ] 里程碑2

## 问题分析

### 待解决问题
1. 问题1
2. 问题2

### 已解决问题
1. 问题 → 方案

## 经验总结

### 有效策略
1. 
2. 

### 待优化项
1. 
2. 

## 下月计划

### 收入目标
- 目标: $__
- 日均目标: $__

### 重点突破
1. 
2. 

### 学习计划
1. 
2. 

---

**自评分数**: __/10

**备注**: 

```

---

## 第三部分：进阶扩展

### schedule.py - 定时运行

```python
#!/usr/bin/env python3
"""
定时任务：每天自动生成日报
用法: crontab -e 添加定时任务
# 每天早上9点运行
0 9 * * * /usr/bin/python3 ~/Projects/monetization-playbook/schedule.py
"""

import sys
sys.path.insert(0, str(__file__).rsplit('/', 1)[0])

from income_tracker import print_daily_report, init_data_files

if __name__ == '__main__':
    init_data_files()
    print_daily_report()
```

### export_to_sheets.py - 导出到Google Sheets

```python
#!/usr/bin/env python3
"""
导出到Google Sheets（高级功能）
需要: pip install gspread google-auth oauth2client
"""
import gspread
from google.auth import default

def export_to_google_sheets(csv_file, sheet_name="收入追踪"):
    """导出CSV到Google Sheets"""
    gc = gspread.authorize(default())
    
    # 创建新表格
    sh = gc.create(sheet_name)
    
    # 上传CSV
    with open(csv_file, 'r') as f:
        content = f.read()
    
    gc.import_csv(sh.id, content)
    print(f"已导出到Google Sheets: {sh.url}")
    return sh.url
```

---

## 快速上手清单

```bash
# 1. 安装
pip install pandas

# 2. 初始化
python income_tracker.py today

# 3. 添加第一笔收入
python income_tracker.py add \
  -t freelance \
  -p fiverr \
  -n "网站爬虫项目" \
  -a 25

# 4. 查看统计
python income_tracker.py stats

# 5. 生成周报
python income_tracker.py report
```

---

**数据是优化的基础。每天记录，月月复盘。**