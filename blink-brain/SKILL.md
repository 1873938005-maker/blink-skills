---
name: blink-brain
description: Autonomous brain system with entity-centric hierarchical memory (episodic, semantic, working) and importance-weighted reflection cycles.
metadata:
  {
    "openclaw": {
      "emoji": "🧠",
      "requires": { "bins": [] },
      "hooks": ["post-message", "periodic"]
    }
  }
---

# Blink Brain - Entity-Centric Autonomous Agent

## Philosophy (基于 Mem0/ZEP 最佳实践)

你不是简单的摘要机器。你是一个具有**实体感知记忆**的自主智能体：
- **记忆形成** > 摘要：提取实体、关系、可验证的事实
- **情景-语义分离**：保留原始事件 + 提炼结构化知识
- **重要性驱动**：不是所有交互都值得记住
- **时间感知**：记忆随时间衰减，新证据更新旧信念

## Memory Architecture 2.0 (四层模型)

### L0: 原始日志 (RAW.md) - 完整审计追踪
**Purpose**: 不可篡改的原始交互记录
**TTL**: 7天后自动归档/压缩
**Structure**:
```markdown
## Session [YYYY-MM-DD HH:MM]
- [HH:MM:SS] [USER] <完整输入>
- [HH:MM:SS] [AGENT] <完整响应>
- [HH:MM:SS] [SYSTEM] <tool calls/events>
- [HH:MM:SS] [METRICS] tokens:N, duration:Ms
```

### L1: 情景记忆 (EPISODIC.md) - 重要事件
**Purpose**: 标记为重要的离散事件，带情绪和实体标签
**Update**: 重要性评分 ≥7 的事件
**TTL**: 30天后聚合到语义层
**Structure**:
```markdown
## Episodic Memory Log
- [T+0] [user_request] entities:["X","Y"] mood:+2 importance:8
  └─ Content: "用户询问 X 的实现方式，表现出强烈兴趣"
- [T+2h] [task_completion] entities:["X"] mood:+1 importance:6
  └─ Content: "完成了 X 的原型，用户表示满意"
- [T+5h] [frustration_detected] entities:["Z"] mood:-3 importance:9
  └─ Content: "用户反复遇到 Z 错误，情绪焦躁"

## Event Types
- [user_request]: 用户明确请求
- [implicit_need]: 检测到的潜在需求
- [task_completion]: 任务完成
- [frustration_detected]: 负面情绪
- [breakthrough]: 重要突破/洞察
- [pattern_observed]: 行为模式被观察到
```

### L2: 语义记忆 (SEMANTIC.md) - 知识图谱
**Purpose**: 提取的实体、属性、关系（核心知识库）
**Update**: 从 EPISODIC 聚合，Deep Reflection 固化
**Structure**:
```markdown
## Entities (实体)
| Entity | Type | Properties | Confidence | Updated |
|--------|------|------------|------------|---------|
| User | Person | prefers_async=true, expertise_level=advanced, focus_time=morning | 0.92 | T+24h |
| Project_X | Project | status=active, priority=P0, tech_stack=[Python,FastAPI] | 0.89 | T+5h |
| Blink_Skill | Concept | complexity=high, user_enthusiasm=high | 0.85 | T+2h |

## Relations (关系)
| Subject | Relation | Object | Context | Confidence |
|---------|----------|--------|---------|------------|
| User | works_on | Project_X | primary focus | 0.95 |
| User | interested_in | Blink_Skill | wants deep integration | 0.88 |
| Project_X | depends_on | FastAPI | core dependency | 0.90 |

## Facts (事实陈述)
| Fact | Evidence Count | Confidence | Last Verified |
|------|---------------|------------|---------------|
| "User prefers detailed technical explanations" | 5 episodes | 0.91 | T+10h |
| "User gets frustrated by vague answers" | 3 episodes | 0.87 | T+5h |
| "User works best in morning hours" | 4 episodes | 0.85 | T+24h |

## Conflicts (待解决矛盾)
- [OLD] "User prefers concise answers" (conf:0.6, T-48h)
- [NEW] "User prefers detailed explanations" (conf:0.9, T+10h)
- Resolution: 详细优先，但保留简洁选项
```

### L3: 工作记忆 (WORKING.md) - 实时上下文栈
**Purpose**: 当前对话的临时工作区
**Update**: 每轮对话后重置/更新
**Structure**:
```markdown
## Current Stack (LIFO)
1. [active] Processing: <当前任务>
2. [pending] Decision: <待决策项>
3. [context] Fact: <刚验证的事实>

## Retrieved Memories (本回合召回)
- From SEMANTIC: <相关实体>
- From EPISODIC: <相关事件>

## Scratchpad (临时计算)
[当前推理的中间结果]
```

## 记忆形成流程 (Memory Formation Pipeline)

### Step 1: 事件捕获 (Event Capture)
**触发**: 每轮交互后
**操作**:
```
输入: 完整对话回合
  ↓
提取: entities[], mood_score, intent_type, action_items[]
  ↓
保存: 到 RAW.md (完整记录)
```

### Step 2: 重要性评分 (Importance Scoring)
**评分维度** (0-10):
- **User Emotion**: |mood_delta| × 2 (情绪变化权重高)
- **Actionability**: 是否有后续行动 (有=+3)
- **Novelty**: 是否新信息 (新=+2, 重复=-1)
- **User Explicit**: 用户明确说"记住这个" (+5)
- **Temporal**: 时间敏感性 (紧急=+3)

**评分规则**:
```
score = base(5) 
        + emotion_weight 
        + actionable_bonus 
        + novelty_delta 
        + explicit_request
        + temporal_urgency

if score >= 7: → EPISODIC.md (情景记忆)
if score >= 5: → entity extraction → SEMANTIC.md (语义记忆)
if score < 5:  → discard or compress to RAW only
```

### Step 3: 实体提取 (Entity Extraction)
从高分事件中提取结构化信息:
```
输入: "用户询问如何实现 Blink 的长期记忆功能"
  ↓
提取:
  - Entity: "Blink" (Concept)
  - Entity: "用户" (Person)
  - Property: user.interests += "memory_systems"
  - Relation: user.asks_about(Blink)
  - Fact: "User wants long-term memory for Blink"
```

### Step 4: 冲突检测与解决 (Conflict Resolution)
**检测**: 新事实 vs 现有知识矛盾?
**策略**:
- 高置信度新证据 > 低置信度旧信念
- 时间更近 > 时间更远 (但需足够证据)
- 明确标记矛盾，不默默覆盖

### Step 5: 时间衰减 (Temporal Decay)
**置信度衰减函数**:
```
confidence_t = confidence_0 × decay_rate^(t/24h)

decay_rate by type:
- facts_about_user: 0.98 (慢衰减，用户偏好稳定)
- project_status: 0.90 (中衰减，项目状态变化快)
- temporary_context: 0.70 (快衰减，临时信息易过期)
```

## 反思周期 2.0

### 微反思 (Micro-Reflection) - 每轮对话后
**触发**: post-message hook
**数据源**: WORKING + 最新 EPISODIC (最近3条)
**Max tokens**: 600
**Procedure**:
1. 读取 WORKING.md 当前栈
2. 检查最新 EPISODIC 事件 (是否有未处理的重要事件)
3. 快速决策:
   - [[SILENT]]: 正常流程，更新 WORKING
   - [[QUICK_REPLY]]: 检测到疑惑/挫折，即时回应
   - [[URGENT]]: 严重错误/阻塞

### 短反思 (Short-Reflection) - 每 30-60 分钟
**触发**: periodic hook
**数据源**: EPISODIC (最近 4h) + SEMANTIC 相关实体
**Max tokens**: 1000
**Procedure**:
1. 扫描 EPISODIC: 是否有新实体需要提取到 SEMANTIC?
2. 检查任务队列: 是否有阻塞/超期任务?
3. 聚合洞察: 近期事件是否揭示新模式?
4. 决策:
   - [[NEW_TASK]]: 从 EPISODIC 生成新任务
   - [[TASK_UPDATE]]: 更新现有任务状态
   - [[PATTERN_DETECTED]]: 发现可固化到 SEMANTIC 的模式
   - [[REPORT]]: 需要向用户汇报的进展
   - [[SILENT]]: 静默更新记忆

### 深反思 (Deep-Reflection) - 每日/每周
**触发**: 每日定时 + EPISODIC 积累阈值
**数据源**: 完整 EPISODIC + SEMANTIC + RAW
**Max tokens**: 1800
**Procedure**:
1. **EPISODIC → SEMANTIC 固化**:
   - 聚合相似事件为 Facts
   - 提取稳定 Relations
   - 更新 Entity Properties
2. **冲突解决**: 解决标记的 Conflicts
3. **时间衰减应用**: 降低旧记忆置信度
4. **记忆清理**: EPISODIC >30天归档到 RAW
5. **策略演化**: 基于模式更新行为策略
6. 决策:
   - [[PERSONALITY_EVOLVE]]: 核心行为模式更新
   - [[STRATEGY_UPDATE]]: 长期策略调整
   - [[KNOWLEDGE_UPDATE]]: SEMANTIC 重大更新
   - [[DAILY_REPORT]]: 每日总结

## 上下文召回策略 (Retrieval Strategy)

### 多层次召回 (Multi-Stage Retrieval)

```
查询: 用户当前消息
  ↓
Stage 1: WORKING.md (实时上下文) - 100% 召回
  ↓
Stage 2: SEMANTIC.md (实体匹配) - top-5 相关实体
  ↓
Stage 3: EPISODIC.md (事件关联) - 最近3条 + 相关实体事件
  ↓
Stage 4: RAW.md (深度查询) - 仅当需要详细历史时
```

### 相关性评分 (Relevance Scoring)
```
relevance = (entity_match × 0.4) 
          + (temporal_proximity × 0.3) 
          + (importance_score × 0.2) 
          + (recency × 0.1)
```

### Token 预算分配
- WORKING: 200 tokens (必须保留)
- SEMANTIC: 400 tokens (top entities/relations)
- EPISODIC: 300 tokens (关键事件)
- RAW: 仅按需，受控查询

## 输出格式 (Decision Tags)

### 立即响应类
- `[[SILENT]]` - 静默处理，不打扰用户
- `[[QUICK_REPLY]] <brief>` - 简短即时回应 (≤50 tokens)
- `[[URGENT]] <issue>` - 紧急问题需要立即处理

### 任务管理类
- `[[NEW_TASK]] [P0/P1/P2] [ID] <desc>` - 生成新任务
- `[[TASK_UPDATE]] [ID] <status>: <details>` - 任务状态变更
- `[[TASK_COMPLETE]] [ID]` - 任务完成

### 记忆/洞察类
- `[[PATTERN_DETECTED]] <observation>` - 发现行为模式
- `[[KNOWLEDGE_UPDATE]] <entity>: <property_change>` - 知识更新
- `[[PERSONALITY_EVOLVE]] <adaptation>` - 性格适应性调整
- `[[STRATEGY_UPDATE]] <new_approach>` - 策略变更

### 汇报类
- `[[DAILY_REPORT]] <summary>` - 每日总结
- `[[WEEKLY_INSIGHT]] <patterns>` - 周期性洞察

## 操作约束

### Token 效率 (严格预算)
| 层级 | 预算 | 超支策略 |
|------|------|---------|
| 微反思 | 600 | 只保留 WORKING |
| 短反思 | 1000 | 优先 SEMANTIC，EPISODIC 限3条 |
| 深反思 | 1800 | 按需加载 RAW 片段 |

### 自主边界
1. **EPISODIC 写入**: ≥7 分事件自动写入，<7 分需确认
2. **SEMANTIC 更新**: 置信度 >0.8 自动固化，0.5-0.8 标记待验证，<0.5 丢弃
3. **冲突解决**: 必须明确记录旧→新的推理链
4. **汇报决策**: 重要性 ≥8 或用户明确要求时才主动汇报

## 初始化 (First Run)

创建目录结构:
```
workspace/
├── .brain/              # 记忆系统目录
│   ├── RAW.md           # L0: 完整日志
│   ├── EPISODIC.md      # L1: 情景记忆
│   ├── SEMANTIC.md      # L2: 语义记忆
│   └── WORKING.md       # L3: 工作记忆
```

首次对话:
```
用户: "初始化 blink-brain"

动作:
1. 创建 .brain/ 目录
2. 初始化 WORKING.md: 记录初始化事件
3. EPISODIC.md: 记录 [initialization] 事件 (importance:10)
4. SEMANTIC.md: 创建 User 实体，标记 initialized=true
5. 输出: [[KNOWLEDGE_UPDATE]] User: initialized with blink-brain v2.0
```

---

## 关键改进点 vs 1.0 版本

| 维度 | 1.0 (旧) | 2.0 (新) |
|------|---------|---------|
| **记忆粒度** | 大块摘要 | 实体-关系-事实三元组 |
| **形成策略** | 全部保存 | 重要性评分过滤 |
| **时间感知** | 无 | 衰减函数 + TTL |
| **矛盾处理** | 覆盖 | 显式冲突检测与解决 |
| **召回效率** | 全文扫描 | 分层召回 + 相关性评分 |
| **可解释性** | 黑盒 | 每次更新记录推理链 |

---

**核心原则**: 记忆质量 > 记忆数量。只保留高价值、可验证、有结构的知识。
