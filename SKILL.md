---
name: stemstar-baoxiao
description: Organize and audit StemStar reimbursement materials for Sun David/StemStar workflows. Use when the user asks to check reimbursement naming, verify whether invoices/receipts/order screenshots/payment screenshots/itineraries are complete, remind them about missing or mismatched amounts, place new materials into the current month's YYMM 待报销 folder, replace generic folders like 待整理 or 6月 with YYMM 待报销, or move submitted items into monthly archived folders named like YYMM 报销-amount.
---

# StemStar Baoxiao Skill

## Purpose

Use this skill to organize reimbursement materials in local and cloud workspaces. The expected workflow is:

1. Put new, unsubmitted materials into a monthly pending folder (local or cloud).
2. Normalize item folders and file names by amount, event, route, and material type.
3. Check amount consistency across invoices, payment screenshots, order screenshots, itineraries, and zip contents.
4. Remind the user about missing invoices, missing itineraries, amount mismatches, unclear payers/buyers, or routes that do not match.
5. After the user says items were submitted, move them into `00 归档/YYMM 报销-总金额`.
6. When the PC is off, accept materials via cloud storage bridge; sync to local NAS on startup.

## Workspace

Default reimbursement root:

`D:\NAS_Amasun\01 工作\教学任务\D01 报销`

If the user gives another reimbursement folder, use that folder. If the current working directory is already inside a reimbursement root, prefer the current root. Never assume the Desktop is the final storage location; files supplied from Desktop should usually be copied into the reimbursement workspace, leaving originals intact unless the user explicitly asks to move/delete them.

## Cloud Storage（云���中转 · 可选）

When the user's PC is offline (e.g., they send materials from mobile while traveling), a cloud knowledge base can serve as a bridge. **Cloud storage is optional — local folder is the primary/required workspace.**

### Recommended: 乐享知识库（Lexiang）

If the user has the Lexiang MCP connector, use it as the cloud bridge:

- **To accept materials offline:** Upload screenshots/PDFs to the user's personal knowledge base in Lexiang (usually named `{姓名}的个人知识库`).
- **Upload flow:** Use Lexiang MCP: `file_apply_upload` → HTTP PUT to `upload_url` → `file_commit_upload` → `entry_create_entry` to place in folder.
- **Folder convention in cloud:** Mirror local structure — create `StemStar 报销中转/YYMM 待报销/` in the knowledge base.
- **On PC startup (sync):** Scan the cloud `StemStar 报销中转` folder → download any new files → archive to local NAS → mark as synced.

### Fallback: ima 知识库

If Lexiang is not connected but ima-mcp is:

- Use `create_media` → COS upload → `add_knowledge` to the user's ima knowledge base.
- Same folder convention: `StemStar 报销中转/YYMM 待报销/`.

### Cloud Setup Dialogue

During first-time setup, after creating the local folder, ask:

> 要不要顺便配个云端中转？电脑关机时，手机发的报销材料可以暂存在云端。
>
> 推荐 **乐享知识库**（如果你接了）：关机也能收材料，开机会自动同步到本地。
>
> 不需要的话跳过，只用本地文件夹也完全够用。

If user says yes and Lexiang is connected:
1. Call `whoami` to get `personal_space.root_entry_id`.
2. Create folder `StemStar 报销中转` inside personal space via `entry_create_entry`.
3. Confirm to user: cloud bridge ready.

If user says yes and only ima is connected, use ima instead.

If user skips, proceed with local-only setup.

## First-Time Setup（首次使用引导）

When the reimbursement root does not exist or is empty (no `00 归档`, no `YYMM 待报销`), this is a new user. Do NOT error out. Instead, guide them through a one-time setup:

### Trigger

Any of these signals mean first-time setup is needed:
- Reimbursement root path does not exist on disk.
- Root exists but is empty (no `00 归档/` and no `* 待报销/`).
- User says "第一次用" / "怎么开始" / "创建报销文件夹".

### Setup Dialogue

1. **Detect the problem.** Internally note that the workspace is missing. Do not show an error.

2. **Ask one question — where to create the folder:**

   > 看起来这是你第一次用报销整理。报销文件夹建在哪儿？
   >
   > 建议：`桌面/报销材料` 或 `文档/报销材料`，以后方便找到就行。
   >
   > 也可以直接告诉我路径，比如 `D:\工作\报销`。

3. **After user confirms the path**, create this minimal structure:

   ```
   报销根目录/
   ├── 00 归档/
   └── YYMM 待报销/    ← 当月，如 2608 待报销
   ```

4. **Confirm to user:**

   > 已建好：
   > - `路径/00 归档/` → 提报后归档
   > - `路径/2608 待报销/` → 当月待报销材料
   >
   > 以后发支付截图、订单截图、发票给我，我帮你归进去。

5. **Remember the path** for this session. Ask user if they want it as permanent default (if they confirm, note it; future sessions will need to re-ask or be configured).

### Existing users (structure already exists)

Skip the setup dialogue. Use the existing workspace as-is. If `00 归档` is missing but other structure exists, create it silently.

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

## Cloud Sync Workflow（云端同步 · 开机自动执行）

When the skill is loaded at session start, silently check if a cloud bridge is configured:

### On Session Start (Skill Load)

1. **Check for cloud config.** If the user has previously set up a cloud storage (stored in workspace memory), quickly verify it's still accessible.
2. **Scan cloud for new materials.** Call the appropriate MCP to list entries in `StemStar 报销中转/`:
   - Lexiang: `entry_list_children` on the bridge folder.
   - ima: `get_knowledge_list` on the bridge folder.
3. **Download and archive.** For any new files found:
   - Download via MCP (`file_download_file` for Lexiang, or `fetch_media_content` for ima).
   - Save to the correct local `YYMM 待报销/金额 事项/` folder.
   - Rename according to file naming rules.
   - Report: "从云端同步了 N 个新材料：{列表}"
4. **No new files.** Say nothing — don't interrupt the user.

### On Receiving Materials While PC is Off

When the user is on mobile and the PC is off:
- If they send screenshots via WorkBuddy cloud, accept them.
- Upload to the configured cloud bridge under `StemStar 报销中转/YYMM 待报销/金额 事项/`.
- Note: "已存到云端中转，电脑开机后自动同步到本地报销文件夹。"

### Skill Update Check

At session start, also check for skill updates:
- Run `git -C <skill_dir> fetch origin main 2>/dev/null && git -C <skill_dir> diff --stat origin/main` to see if updates are available.
- If newer version exists, mention briefly: "stemstar-baoxiao 有新版本，已自动拉取" and `git pull`.
- If pull fails (no network, no git), skip silently.

## Safety

- Before recursive moves, verify source paths are under the reimbursement root and archive destinations are under `00 归档`.
- Do not overwrite existing folders or files. If a destination exists, inspect and choose a non-destructive path or ask the user.
- Do not use broad destructive cleanup. Deleting raw reimbursement files is out of scope unless explicitly requested.
- Preserve unrelated folders.
- Keep the reimbursement root easy to scan: normally only `00 归档`, one or more `YYMM 待报销` folders, and reference files such as `报销细则.pptx` should remain there.
- Keep final summaries concise: state the new path, total, reconciliation equation, important mismatches, and pending gaps.
