# Ikasan Java 规范（团队补充）

## 代码风格
- 4 空格缩进，大括号另起一行（Allman 风格），和现有代码保持一致
- 类名 PascalCase

## 异常处理
- 禁止空 catch
- 组件（Converter / Filter / Router 等）内部捕获异常后，包装为 Ikasan spec 中对应的异常（如 TransformationException）向上抛出，
  交给框架的 error-reporting / exclusion 处理
- 组件内不要"记录日志后吞掉异常"，也不要在抛出前重复记录日志

## 日志
- 使用 slf4j（org.slf4j.Logger / LoggerFactory）
- 日志级别最高 INFO（CI 日志有 4MB 上限）