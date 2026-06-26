# neo-write-skill 验收标准

> Loop QA 验收文件。Claude 在 `/loop-qa neo-write-skill` 时读这个文件，逐项检查。

## 基本信息
- **验收对象**: skill 输出的文字内容质量
- **自动化程度**: 中（AI 腔检测可自动化，声音像不像需人工）
- **前置**: `memory/voice-profile.md` 必须非空

---

## 测试场景

### S1: 朋友圈（最短、最私人）

**输入**: 一段碎片想法（1-3 句话）
**操作**: 调用 `/nws`，指定平台=朋友圈
**期望输出**: 30-150 字，第一行即钩子，无标题

**检查清单**:
- [ ] `AUTO` voice-profile.md 已读取且非空
- [ ] `AUTO` AI 腔黑名单检查通过（跑下方脚本）
- [ ] `AUTO` 字数 30-150
- [ ] `AUTO` 无超过 6 行（会被折叠）
- [ ] `MANUAL` 第一句有钩子（不是废话开头）
- [ ] `MANUAL` 语气最松弛——像跟朋友说话
- [ ] `MANUAL` 不像广告、不像小作文
- [ ] `MANUAL` 读起来像用户本人写的（≥ 4/5 分）

**通过条件**: 自动检查全过 + 像不像 ≥ 4 分

---

### S2: 小红书文案（核心场景）

**输入**: 一个观点/经历/方法论
**操作**: 调用 `/nws`，指定平台=小红书
**期望输出**: 标题 ≤ 20 字 + 正文 300-800 字 + 5-10 个话题标签

**检查清单**:
- [ ] `AUTO` AI 腔黑名单通过
- [ ] `AUTO` 标题字数 ≤ 20
- [ ] `AUTO` 正文字数 300-800
- [ ] `AUTO` 话题标签 5-10 个（`#` 开头）
- [ ] `AUTO` 正文有足够换行（平均段落 ≤ 4 行）
- [ ] `MANUAL` 标题有钩子（数字/痛点/反差/身份）
- [ ] `MANUAL` 开头 1-2 行留住人
- [ ] `MANUAL` 结构清晰（痛点→解法→收口 / 故事→转折→学到 / 反常识清单）
- [ ] `MANUAL` 知识付费场景：有真内容 + 软钩子，非硬广
- [ ] `MANUAL` 像用户本人写的 ≥ 4/5 分

**通过条件**: 自动检查全过 + 结构清晰 + 像不像 ≥ 4 分

---

### S3: 推特/X（最锐最短）

**输入**: 一个观点
**操作**: 调用 `/nws`，指定平台=推特
**期望输出**: 单条 ≤ 140 中文字 或 Thread

**检查清单**:
- [ ] `AUTO` AI 腔黑名单通过
- [ ] `AUTO` 单条 ≤ 140 字
- [ ] `AUTO` 无小红书腔（满屏 emoji、「家人们」）
- [ ] `MANUAL` 锐、快、敢站队
- [ ] `MANUAL` 短句优先

---

### S4: 公众号长文

**输入**: 一个主题 + 要讲透的核心观点
**操作**: 调用 `/nws`，指定平台=公众号
**期望输出**: 1500-4000 字长文

**检查清单**:
- [ ] `AUTO` AI 腔黑名单通过
- [ ] `AUTO` 字数 1500-4000
- [ ] `AUTO` 段落长短交错（不是每段一样长）
- [ ] `MANUAL` 开头有具体场景/故事（不是抽象感悟）
- [ ] `MANUAL` 有真实毛边（「我也没完全想通」类句子）
- [ ] `MANUAL` 结尾不是「希望对你有帮助」

---

### S5: 学习回路验证

**输入**: 在 S1-S4 任一场景定稿后
**操作**: 检查 voice-profile.md 是否更新
**检查清单**:
- [ ] `AUTO` voice-profile.md 的修改时间 > 定稿时间（说明写回了）
- [ ] `MANUAL` 学习日志追加了新条目（带日期）
- [ ] `MANUAL` 如果学习日志 ≥ 15 条，有合并消化的痕迹

**通过条件**: 写回动作发生

---

## 自动检查脚本

### AI 腔黑名单检测（用 grep 跑）

```bash
#!/bin/bash
# 用法: bash qa-check-write.sh output.txt
# 返回 0 = 通过, 1 = 有命中

FILE="$1"
if [ -z "$FILE" ]; then echo "用法: bash qa-check-write.sh <output.txt>"; exit 2; fi

HITS=0
echo "=== NEO WRITE QA: AI 腔检测 ==="

# 连接词病
for WORD in "首先" "其次" "再次" "总而言之" "综上所述" "总的来说"; do
  if grep -q "$WORD" "$FILE"; then
    echo "  ✗ 连接词病: 「$WORD」"
    ((HITS++))
  fi
done

# 万能开头
for WORD in "在这个.*的时代" "随着.*的发展" "众所周知" "不得不说"; do
  if grep -qE "$WORD" "$FILE"; then
    echo "  ✗ 万能开头: 「$WORD」"
    ((HITS++))
  fi
done

# 商业黑话
for WORD in "赋能" "抓手" "闭环" "对齐" "颗粒度" "打法" "心智" "势能"; do
  if grep -q "$WORD" "$FILE"; then
    echo "  ✗ 商业黑话: 「$WORD」"
    ((HITS++))
  fi
done

# 空心强调
for WORD in "满满的干货" "纯纯的" "真的真的" "绝绝子"; do
  if grep -q "$WORD" "$FILE"; then
    echo "  ✗ 空心强调: 「$WORD」"
    ((HITS++))
  fi
done

# 结尾病
for WORD in "希望对你有帮助" "你学会了吗" "快去试试吧" "记得点赞收藏关注" "我们下期再见"; do
  if grep -q "$WORD" "$FILE"; then
    echo "  ✗ 结尾病: 「$WORD」"
    ((HITS++))
  fi
done

# 对偶病（高频滥用）
PAIRS=$(grep -cE "不仅.*更是|既要.*又要|不是.*而是" "$FILE" 2>/dev/null || echo 0)
if [ "$PAIRS" -gt 1 ]; then
  echo "  ✗ 对偶病: 出现 ${PAIRS} 次（偶尔用可以，多次 = AI 腔）"
  ((HITS++))
fi

# 列表病：连续 4+ 行用 emoji 开头
EMOJI_LINES=$(grep -cE "^[✅🔥💡⭐🎯📌✨🚀💪❤️]" "$FILE" 2>/dev/null || echo 0)
if [ "$EMOJI_LINES" -gt 3 ]; then
  echo "  ✗ 列表病: ${EMOJI_LINES} 行 emoji 开头"
  ((HITS++))
fi

if [ "$HITS" -eq 0 ]; then
  echo "  ✅ AI 腔检测通过（0 命中）"
  exit 0
else
  echo "  ❌ AI 腔检测失败（${HITS} 处命中）"
  exit 1
fi
```

### 平台格式检测（Claude 内部执行）

Claude 在验收时直接计算:
1. 字数 = 去掉标签后的中文+英文字符数
2. 段落数 = 按空行分割后的段落数
3. 段落均长 = 总字数 / 段落数
4. 段落长度方差 = 是否每段一样长（方差太小 = AI 的均匀病）

---

## 常见失败 & 修复

| 失败 | 原因 | 修复 |
|------|------|------|
| AI 腔命中 | 默认语言习惯带出 | 逐个替换为声音档案中用户会用的表达 |
| 太均匀 | 每段 3 行、每个观点配 3 个例子 | 刻意让段落长短交错，有的观点不给例子 |
| 不像用户 | 没读 voice-profile 或读了没用 | Step 0 强制读档案，Step 3 对照 samples 写 |
| 开头太空 | 用了「在这个时代」类万能开头 | 从具体场景/一件事切入 |
| 结尾太油 | 号召式收尾 | 戛然而止或留一个真实的疑问 |
| 字数不对 | 没看 platforms.md 的平台规范 | Step 2 确认平台后先读规范 |
| 没写回档案 | 跳过了 Step 5 | 每次定稿后必须执行学习回路 |

---

## Loop 使用方式

```bash
# 单次验收
/loop-qa neo-write-skill

# 持续改进声音像度
/loop 用 neo-write-skill 给这段想法写小红书文案，写完跑 QA.md 的 AI 腔检测，如果命中就改到 0 命中为止

# 校准声音档案
/loop 读 memory/samples/ 里最新 3 篇样本，对照 voice-profile.md 检查有没有过时条目，更新后用一段测试内容验证像度
```
