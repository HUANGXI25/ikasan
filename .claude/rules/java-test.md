---
paths:
  - "**/*Test.java"
---
# 测试规范
- 测试方法名用 should_结果_when_条件 的格式（练习用规则，与现有 testXxx 写法不同）
- 每个测试方法只验证一个行为
- JUnit 版本与所在模块现有测试保持一致（converter 模块是 JUnit 4）