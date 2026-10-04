# DONE

## 2026-10-04 — mkmaciso + README 中英文双语（PR #1）
- 改动：mkmaciso 加语言检测（--lang > MKMACISO_LANG > 自动检测 > 默认 en）+ load_strings()（130+ MSG_* 变量），全量用户可见文案变量化，printf 风格插值；README 中文在前英文在后完整翻译。
- 验证：bash -n 通过；--help 中英文输出正常，边框对齐，无未展开变量；--lang zh/--lang=en/MKMACISO_LANG/LANG 自动检测均验证。
- 顺手修复：--help tagline 行尾多余 `"`；磁盘空间警告缺右括号。
- 状态：PR #1 已开，未合并，分支保留。
