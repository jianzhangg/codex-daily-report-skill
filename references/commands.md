# Commands

下面这些命令是“根据本地 Codex 会话生成当日工作日报”的通用骨架。

占位说明：

- `{date}`：形如 `2026-04-20`
- `{cwd}`：目标工作区根目录
- `{rollout_path}`：某个线程对应的 `jsonl` 文件路径

## 1. 如果用户没明确工作区，先看当天有哪些 `cwd`

```sh
sqlite3 ~/.codex/state_5.sqlite \
  "select distinct cwd
   from threads
   where date(updated_at,'unixepoch','localtime')='{date}'
   order by cwd;"
```

## 2. 查当天指定工作区的线程

如果用户给的是工作区根目录，建议把子目录线程一起带上：

```sh
sqlite3 ~/.codex/state_5.sqlite \
  "select id, title, datetime(updated_at,'unixepoch','localtime') as updated_local, cwd
   from threads
   where date(updated_at,'unixepoch','localtime')='{date}'
     and (cwd='{cwd}' or cwd like '{cwd}/%')
   order by updated_at;"
```

## 3. 查线程数量

```sh
sqlite3 ~/.codex/state_5.sqlite \
  "select count(*)
   from threads
   where date(updated_at,'unixepoch','localtime')='{date}'
     and (cwd='{cwd}' or cwd like '{cwd}/%');"
```

## 4. 查每条线程对应的 rollout 文件

```sh
sqlite3 ~/.codex/state_5.sqlite \
  "select id, rollout_path
   from threads
   where date(updated_at,'unixepoch','localtime')='{date}'
     and (cwd='{cwd}' or cwd like '{cwd}/%')
   order by updated_at;"
```

## 5. 从单个线程里抽目标日期的用户消息

```sh
jq -r '
  select(.type=="event_msg"
    and .payload.type=="user_message"
    and (.timestamp|startswith("{date}")))
  | [.timestamp, .payload.message]
  | @tsv
' "{rollout_path}"
```

## 6. 查当前工作区里是否已有日报目录约定

在工作区根目录执行：

```sh
find . -maxdepth 4 -type d \
  \( -name daily -o -name report -o -name reports -o -name 日报 \) \
  | sort
```

如果要顺手看已有日报文件：

```sh
find . -maxdepth 5 -type f -name "*.md" \
  | rg "/(daily|report|reports|日报)/"
```

## 7. 输出路径规则

优先级：

1. 用户明确给了文件路径或目录，就直接用。
2. 用户没给时，先沿用当前工作区里已存在的日报目录约定。
3. 如果没有明显约定，默认回退到当前工作区下的 `./daily/{date}.md`。

如果目标文件已存在，默认先读原文件，再补充或调整，不要直接覆盖。

## 8. 使用建议

- 先看线程标题和更新时间，初步判断哪些像是真工作。
- 再用第 5 步逐条抽用户消息，确认线程实际内容，不要只看标题。
- 线程跨天时，以内部 `timestamp` 是否属于目标日期为准，不看文件修改时间。
- “写日报”这条当前线程要排除，不要算进当天工作。
- 生成日报前，可以再结合当天代码、文件或提交情况做辅助核对，但会话记录仍然是主依据。

## 9. 归并思路

优先按下面维度归并：

- 一个需求或一条方案主线
- 一个独立 bug 的排查或修复
- 一条完整的接入、联调、验证或上线准备工作
- 一组连续的业务理解或文档梳理工作

不要按下面维度归并：

- 上午做了什么、下午做了什么
- 开了几个线程
- 查了几个页面或跑了几次命令
