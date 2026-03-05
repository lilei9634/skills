---
name: amazon-ads-report-audit
description: Validate Amazon Ads and business report bundles for one or multiple shops using a fixed checklist. Use when a user asks to check whether downloaded reports are complete, whether time-granularity is daily, whether files are empty, and what is missing before analysis. 输出语言固定为中文。
---

# Amazon Ads Report Audit

## 输出语言
固定使用中文输出审计结果。
文件名和报表关键字保持原样，不翻译文件名。

## Overview
Use this skill to audit a shop folder and return a strict readiness verdict for ad analysis.

Audit goals:
- Confirm required reports exist.
- Enforce daily-granularity for date-based reports.
- Flag empty files and non-blocking caveats.
- Return missing/non-compliant files in a fixed, concise format.

概述：
使用该技能审核店铺报表目录，并给出分析可用性的明确结论。

审核目标：
- 检查必需报表是否齐全。
- 强制检查时间粒度是否为日级。
- 标记空表与非阻塞风险项。
- 按固定格式输出缺失项与不合规项。

## Inputs
Require these inputs from the user:
- Shop folder path (example: `/Users/.../店铺名/广告数据`).
- Time window expected (default `30 days`).
- Scope note if needed (example: `US only`).

If the user does not provide a time window, assume `30 days`.

## Required Reports (v2)
Check these report name patterns (substring match is acceptable):

1. `广告活动`
2. `Targeting`
3. `商品`
4. `搜索词展示量份额`
5. `搜索结果首页首位展示量份额`
6. `品牌推广关键词`
7. `品牌推广搜索词`
8. `品牌推广归因于广告的购买`
9. `商品推广预算`
10. `商品推广广告位`
11. `商品推广搜索词报告`
12. `商品推广按时间查看效果`
13. `商品推广推广的商品`
14. `展示型推广广告活动`
15. `展示型推广投放`
16. `展示型推广推广的商品`
17. `展示型推广匹配的目标`
18. `展示型推广已购买商品`

## Optional but Recommended
Report as optional status only:
- `品牌推广总流量和无效流量`
- `展示型推广总流量和无效流量`
- `商品推广总流量和无效流量`
- `广告资源`
- `受众`

## Daily-Granularity Hard Rules
For any report that has time fields, only accept daily-level files.

Accept:
- Has `日期` or `Date` as single-day rows.

Reject (non-compliant):
- Only `日期范围`.
- Only `开始日期 + 结束日期` interval rows.

Tie-breaker rule:
- If both daily and interval versions exist for the same report, keep daily and mark interval as ignored.

## Empty File Rules
- Empty required file: mark as `ISSUE`.
- Empty optional file: mark as `WARN` (can be acceptable for no-delivery periods).

## Audit Procedure
1. List files under target folder (`csv`/`xlsx`).
2. Count files by type.
3. Validate required checklist coverage.
4. Read headers and detect date-granularity type.
5. Detect empty files.
6. Return final verdict with action items.

Use quick parsing logic:
- For CSV: `utf-8-sig` first, fallback encoding only if needed.
- For XLSX: read first sheet unless user specifies otherwise.

## Output Format
Return results in this fixed structure:

1. `Summary`
- total files, csv count, xlsx count.

2. `Checklist`
- required: OK/MISSING per item.

3. `Granularity Issues`
- list non-compliant interval files.

4. `Empty Files`
- list empty files and severity (`ISSUE` or `WARN`).

5. `Verdict`
- one of: `Ready`, `Ready with caveats`, `Not ready`.

6. `Next Actions`
- concrete re-download list (only missing/non-compliant items).

## Decision Rules for Verdict
- `Ready`: all required reports present, daily-compliant, no empty required files.
- `Ready with caveats`: all required reports present and daily-compliant, but optional files are empty/missing.
- `Not ready`: any required report missing, non-daily, or empty.

## Multi-shop Mode
If user provides multiple shop folders:
- Run the same checks per shop.
- Return one compact verdict block per shop.
- End with a cross-shop summary table: `shop | ready_status | missing_count | non_daily_count | empty_required_count`.
