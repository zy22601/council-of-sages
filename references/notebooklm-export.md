# 导出智者议会会议到 NotebookLM 生成图片

## 适用场景

当用户希望把智者议会会议的完整记录导出为视觉化信息图时使用。已验证可用于：
- 信息图 (infographic) — PNG 格式
- 幻灯片 (slide deck) — PDF/PPTX
- 视频 (video) — MP4

## 前置条件

- `notebooklm` CLI 已安装并认证（见 notebooklm skill）
- notebooklm 语言已设置为 `zh_Hans`（`notebooklm language set zh_Hans`）

## 工作流（5-8 分钟）

### 1. 保存会议记录为文本文件

```bash
# 将会议记录写到临时文件
cat > /tmp/council-transcript.txt << 'HEREDOC'
[完整的会议记录文本]
HEREDOC
```

### 2. 创建笔记本并上传源

```bash
NB=$(notebooklm create "智者议会：[问题摘要]" --json | python3 -c "import sys,json; print(json.load(sys.stdin)['notebook']['id'])")
SRC=$(notebooklm source add /tmp/council-transcript.txt --json | python3 -c "import sys,json; print(json.load(sys.stdin)['source']['id'])")
notebooklm source wait $SRC --timeout 60
```

### 3. 生成信息图

```bash
# 推荐参数：detailed + professional（适合思想辩论内容）
notebooklm generate infographic \
  "[一句话概括会议主题和核心张力]" \
  --detail detailed \
  --style professional \
  --json

# 风格选项：sketch-note（手绘风）、bento-grid（模块化）、
#   scientific（学术风）、editorial（杂志风）、anime、clay 等
```

### 4. 等待完成并下载

```bash
# 等待生成（通常 2-5 分钟）
notebooklm artifact wait <artifact_id> --timeout 600

# 下载 PNG
notebooklm download infographic ./council-infographic.png -a <artifact_id>
```

## 已验证案例

会议主题：「上帝存在吗」，四位智者：释迦牟尼、庄子、爱因斯坦、慧能
生成参数：`--detail detailed --style professional`
输出：4.8MB PNG 信息图，包含四人核心立场、辩论交锋、智慧提炼

## 提示词技巧

生成信息图时的提示词应包含：
- 会议主题的一句话概括
- 每位智者的核心立场（用风格DNA对应的关键词）
- 最激烈的辩论交锋点
- 轴心张力的命名

避免在提示词中放完整发言文本——NotebookLM 源文件已经有全文。提示词只做「聚焦」。
