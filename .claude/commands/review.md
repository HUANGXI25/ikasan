---
description: 按 Ikasan 团队规范审查指定 Java 文件（只读）
argument-hint: <文件路径>
allowed-tools: Read, Grep, Glob
---
请审查 $ARGUMENTS。只读代码，不要修改任何文件。

审查依据：CLAUDE.md 和 docs-claude/java-style.md。

检查项：
1. 潜在 bug（空指针、类型转换、资源未关闭）
2. 异常处理：是否包装为 Ikasan spec 异常抛出；是否保留了原始异常（cause）；有没有空 catch、宽泛的 catch (Exception)、吞掉异常
3. 日志：是否用 slf4j；有没有超过 INFO 的级别或重复记录日志
4. 和同目录下其他类的写法是否一致

用表格输出：行号 / 问题 / 严重度（高・中・低）/ 修改建议
最后用一句话总结这个文件整体的质量。