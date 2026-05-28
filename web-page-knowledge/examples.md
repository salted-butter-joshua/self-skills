# Usage Examples

## Example 1：单 URL 知识提取

**用户：** 帮我总结这个页面 https://example.com/docs/quickstart

**Agent：**

1. WebFetch URL
2. 输出总览 + 3 个分块（安装 / 配置 / 运行）
3. 每块按 是什么/为什么/怎么做 写总结
4. 生成 12 张记忆卡片
5. 保存 `~/Documents/notes/web-knowledge/20260527--web-knowledge-quickstart.md`

## Example 2：多 URL

**用户：** 这两个链接一起总结：https://a.com/part1 https://a.com/part2

**Agent：** 分别抓取 → 合并总览（标注来源 URL）→ 统一分块编号 B1…Bn → 一份汇编文件。

## Example 3：抓取失败后粘贴

**用户：** 页面抓不下来，正文如下：…

**Agent：** 跳过 WebFetch，在总览中注明「来源：用户粘贴」→ 其余流程相同。

## Example 4：总结 + 视觉卡片

**用户：** 总结这个链接并铸成阅读卡

**Agent：** 完成本 skill → 将第三节+第四节交给 `ljg-card -l`。
