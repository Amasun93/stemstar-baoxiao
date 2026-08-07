---
name: stemstar-baoxiao
description: Organize and audit StemStar reimbursement materials for Sun David/StemStar workflows. Use when the user asks to check reimbursement naming, verify whether invoices/receipts/order screenshots/payment screenshots/itineraries are complete, remind them about missing or mismatched amounts, place new materials into the current month's YYMM 待报销 folder, replace generic folders like 待整理 or 6月 with YYMM 待报销, or move submitted items into monthly archived folders named like YYMM 报销-amount.
---

# StemStar Baoxiao Skill

## Purpose

Use this skill to organize reimbursement materials in the local reimbursement workspace. The expected workflow is:

1. Put new, unsubmitted materials into a monthly pending folder.
2. Normalize item folders and file names by amount, event, route, and material type.
3. Check amount consistency across invoices, payment screenshots, order screenshots, itineraries, and zip contents.
4. Remind the user about missing invoices, missing itineraries, amount mismatches, unclear payers/buyers, or routes that do not match.
5. After the user says items were submitted, move them into `00 归档/YYMM 报销-总金额`.

## Workspace

Default reimbursement root:

`D:\NAS_Amasun\01 工作\教学任务\D01 报销`

If the user gives another reimbursement folder, use that folder. If the current working directory is already inside a reimbursement root, prefer the current root. Never assume the Desktop is the final storage location; files supplied from Desktop should usually be copied into the reimbursement workspace, leaving originals intact unless the user explicitly asks to move/delete them.

## Folder Rules

Use these top-level folders:

- `YYMM 待报销`: current month materials not yet submitted.
- `00 归档/YYMM 报销-总金额`: materials already submitted for that month.

Do not use generic root-level folders such as `待整理`, `待处理`, or plain month names such as `6月`. If they already exist, migrate their contents into the appropriate `YYMM 待报销` folder and remove the empty legacy folder. If a material's month is unclear, put it under the current `YYMM 待报销/待确认材料` rather than creating root-level `待整理`.

Use two-digit year and month, for example:

- `2607 待报销`
- `2606 报销-2334.49`
- `2511 报销-5449.50`
- `2607 待报销/待确认材料`

Inside a monthly folder, group each reimbursement item as:

`金额 事项名称`

Examples:

- `1192.49 武汉讲座`
- `818 南昌讲座`
- `324 半年会交通`

Inside an item folder, create one or more detail folders when useful:

- `1010 武汉讲座机票`
- `182.49 武汉讲座打车`
- `324 半年会火车`

Keep item and detail folder amounts based on the reimbursement amount, usually the actual paid amount when payment screenshots prove it. If the invoice face amount differs, preserve that fact in the file name.

## File Naming

Use this shape:

`金额 事项_对象或路线_材料类型.ext`

Common material types:

- `发票`
- `支付截图`
- `订单截图`
- `行程单`
- `报销明细`

Examples:

- `550 武汉讲座机票_上海-武汉_发票.pdf`
- `460 武汉讲座机票_武汉-上海_发票(票面501).pdf`
- `92.29 武汉讲座打车_武汉大学-天河机场_行程单(票面77.29).pdf`
- `324 半年会火车_订单截图.jpg`

Use the route visible in the invoice, itinerary, or screenshot. Prefer concise route names such as `上海虹桥-南京南`, `武汉大学-天河机场`, or `南昌西-杭州东`.

## Amount Rules

Before moving or renaming, inspect the available evidence:

- PDFs: extract text and read invoice amount, buyer name, date, route, and material type.
- Images: visually inspect screenshots when names do not fully explain the amount or route.
- ZIP files: extract to a temp folder, inspect contents, then copy relevant files into the reimbursement workspace.
- Spreadsheets: read totals when folder names are unclear.

Use exact decimal arithmetic for totals. Do not round away cents.

Always reconcile at least these totals before saying materials are ready:

- Item folder amount.
- Detail folder amount.
- Invoice total.
- Payment screenshot total.
- Order screenshot total.
- Itinerary total when available.

If any total differs, report the exact equation and gap. Example: `返程支付截图 415 = 发票 284 + 87 + 缺口 44`, so the user knows which material to ask for.

When invoice amount and paid amount differ:

- Use the user's stated reimbursement basis if provided.
- Otherwise prefer actual paid amount when a payment screenshot clearly proves it.
- Keep invoice face amount in parentheses, for example `(票面501)` or `(票面77.29)`.
- Mention the mismatch in the final summary.

When a material is missing:

- Keep the item in the correct pending folder if the user is still preparing it.
- Add `待补发票`, `待补行程单`, or similar only when it helps the user see the gap.
- Put ambiguous materials in `YYMM 待报销/待确认材料`, not in a root-level `待整理` folder.
- Once the missing material is supplied, rename the existing placeholder file to remove `待补`.

## Check And Reminder Workflow

For every reimbursement item, produce a short audit result before finalizing:

1. State whether the item is complete or incomplete.
2. List present evidence by material type, such as invoice, payment screenshot, order screenshot, and itinerary.
3. Show the amount reconciliation in one line.
4. Call out missing materials in direct language, for example `缺 44 元火车票发票`.
5. If the user has already submitted the item, still report the issue so they can follow up with the recipient.

Do not silently archive or mark an item as ready when the payment/order total is higher than the invoice total and the difference is unexplained.

## Pending Workflow

When the user supplies new materials and asks to organize them:

1. Identify the month from invoice dates, travel dates, payment dates, or user context.
2. Create or reuse `YYMM 待报销`.
3. If root-level `待整理` or a plain month folder contains pending materials, migrate those materials into `YYMM 待报销`; remove the legacy folder only after confirming it is empty.
4. Create item folders using `金额 事项名称`.
5. Create detail folders if there are multiple categories, such as train, flight, taxi, hotel, software, or meals.
6. Copy supplied Desktop or attachment files into the appropriate folder; do not delete originals.
7. Rename files according to the naming rules.
8. Report the folder path, total amount, reconciliation result, mismatches, and missing materials.

## Submitted Archive Workflow

When the user says an item has been submitted or asks to archive it:

1. Identify the submitted item folders.
2. Compute the submitted total from item folder names and verified evidence.
3. Find an existing archive folder for the same month matching `YYMM 报销-*`.
4. If no archive folder exists, create `00 归档/YYMM 报销-总金额`.
5. If an archive folder exists, move the submitted item into it and rename the archive folder so the total equals old total plus submitted total.
6. Leave unsubmitted items in `YYMM 待报销`.
7. Report the new archive path, reconciliation result, missing material reminders, and what remains pending.

If multiple archive folders exist for the same `YYMM`, inspect before merging. If there are duplicate item names, stop and ask the user which version to keep.

## Historical Cleanup Workflow

When the user asks to clean old archives:

1. Work inside `00 归档`.
2. Normalize roots to `YYMM 报销-金额` when month and total are reliable.
3. Merge multiple submitted folders from the same month into one monthly folder and update the total.
4. Preserve old nested item folders unless the user asks to rename files inside them.
5. Move empty or ambiguous legacy folders to a clearly named `待确认材料` folder inside the relevant archive or pending month; do not create root-level `待整理`.
6. Summarize which folders were normalized and which need manual confirmation.

## Safety

- Before recursive moves, verify source paths are under the reimbursement root and archive destinations are under `00 归档`.
- Do not overwrite existing folders or files. If a destination exists, inspect and choose a non-destructive path or ask the user.
- Do not use broad destructive cleanup. Deleting raw reimbursement files is out of scope unless explicitly requested.
- Preserve unrelated folders.
- Keep the reimbursement root easy to scan: normally only `00 归档`, one or more `YYMM 待报销` folders, and reference files such as `报销细则.pptx` should remain there.
- Keep final summaries concise: state the new path, total, reconciliation equation, important mismatches, and pending gaps.
