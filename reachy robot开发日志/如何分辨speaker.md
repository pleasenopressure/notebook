# 如何分辨 speaker

> 本文记录项目中 speaker identification（发言者身份识别）方案的演进。内容以当前 worktree `baby-reachy-mini-companion` 的实际代码为准，重点区分“曾经尝试过的方案”“当前仍保留的实验模式”和“目前实际采用的方案”。

## 目标与问题边界

这个项目不是只做语音转文字，还必须回答两个不同问题：

1. **这段话是谁说的？** 结果用于给 transcript 标注 `Jack`、`Mary` 或未知 speaker，并送入 `MeetingStateTracker` 统计发言时长、轮次和参与平衡。
2. **机器人现在应该看向谁？** 这是实时注意力动作，要求比完整 STT 更快，不能等一句话识别结束后才转头。

这两个问题有关，但不应该强行使用同一条慢速链路。当前实现将它们拆成：

- 身份链路：麦克风设备绑定身份，最终形成 `SpeakerAssignment`；
- 快速朝向链路：每个已绑定麦克风上的 VAD、RMS 和 SNR 直接选择临时注视目标；
- 语义介入链路：完整 transcript 进入 meeting-state 和 mediation，不负责实时追头。

## 阶段一：通过声纹相似度区分

最初设想是为每位参与者录制 voice baseline，再把新音频与基线进行相似度比较。这一方向的问题包括：

- 短句、噪声和麦克风距离会显著改变声学特征；
- 两个人同时说话时，一段混合音频很难稳定归属于单一说话人；
- 需要维护注册音频、阈值和失败回退，实验成本高；
- 即使声纹最终判对，也通常要等语音段结束，不适合驱动低延迟的“正在看谁”。

**当前状态：已从产品路径移除。** `local/handler.py` 中相关兼容入口只返回 `voiceprint_speaker_id_removed`，不能把日志中的 `voice_baseline` 字段误解成当前仍在执行声纹识别。

## 阶段二：通过限制输入设备物理区分

相比从混合音频中推断身份，更直接的思路是让输入源本身携带身份：一个输入设备固定对应一个参与者。

这一思路最终发展成当前的 `multi_mic` 方案。它的优势是身份来自明确配置，不需要根据声音特征猜测；但仍需要处理串音、环境噪声和两人重叠讲话，不能简单地把“某个麦克风收到声音”直接等同于“该人正在说话”。

## 阶段三：采用 DoA，通过方向分类

参考：[Reachy Mini sound DoA 示例](https://huggingface.co/docs/reachy_mini/examples/sound_doa)

### 3.1 当前代码中的 DoA 数据流

DoA 数据并不是从 STT 返回，而是由 Reachy daemon 的 `/api/state/doa` 独立提供：

```text
Reachy daemon /api/state/doa
        ↓（默认约 10 Hz 轮询）
console.py::doa_loop()
        ↓
handler.observe_doa(angle, speech_detected, monotonic_time)
        ↓
PositionSpeakerIdentifier 保存近期样本，并在一个语音段内收集 DoA
        ↓
语音段结束后 _run_pipeline(audio, doa_samples)
        ↓
classify_with_recent() → SpeakerAssignment → STT transcript 标注 speaker
```

核心实现位于：

- `src/reachy_mini_conversation_app/console.py`：轮询 daemon DoA，并把样本交给 handler；
- `src/reachy_mini_conversation_app/local/position_speaker_id.py`：校准、稳健统计和 speaker 分类；
- `src/reachy_mini_conversation_app/local/handler.py`：将分类结果接入 STT → meeting state → LLM/TTS 主链路。

### 3.2 校准如何工作

每位参与者先在固定座位上完成校准。`PositionSpeakerIdentifier` 从实时窗口中取有效 DoA 样本，以角度中位数作为该人的参考方向，以 MAD（median absolute deviation）衡量角度是否稳定。

当前默认条件包括：

- 至少 8 个有效样本；
- 至少持续 2 秒；
- 最长等待 10 秒；
- 校准 MAD 不得大于 0.20 rad；
- 两位参与者的参考方向至少相隔 0.35 rad。

若时长不足、样本不足、角度波动太大或两人的方向太接近，校准会被拒绝。DoA 模式下，只要已配置的参与者尚未全部校准，`_run_pipeline()` 就会记录 `conversation_blocked_calibration_incomplete` 并暂不处理该段对话，以避免把未知身份写入参与平衡统计。

### 3.3 一句话如何匹配到参与者

一句话结束后，分类器先计算该段有效 DoA 的中位数和 MAD，再寻找距离最近的已校准方向。当前默认保护条件是：

- 至少 3 个有效样本；不足时可从最近 1.5 秒窗口补样本；
- 该段 MAD 不得大于 0.25 rad；
- 与最近参考方向的距离不得大于 0.35 rad；
- 最近与第二近候选之间的 margin 至少为 0.15 rad。

条件不满足时返回 `unknown` 或 `uncertain`，而不是强行把声音分给某个人。日志中的 `doa_sample_summary` 和 `position_match` 会保存中位角、MAD、样本数、补样本情况、匹配距离、margin 和失败原因，便于解释每次结果。

### 3.4 衍生问题：DoA 直接驱动头部跟随过于敏感

最初把每个瞬时 DoA 角度直接用于机器人转头，实际出现安静房间内仍不断抖动的问题。原因包括反射、回声、瞬时误检以及机器人自己运动后坐标关系变化。

代码曾通过以下限制降低敏感度：

- 只有 `speech_detected=true` 时才考虑跟随；
- 转角变化至少达到阈值才动作（默认约 30°）；
- 限制最大 yaw（默认约 60°）；
- 用语音段内多个样本的统计值识别 speaker，而不是信任单点角度。

但这些保护只能缓解抖动，不能消除“定位坐标系随机器人转头变化”的根本问题。

### 3.5 speaker 跟随与 calibration 的冲突

校准假设机器人在一个稳定姿态下测量“参与者相对于麦克风阵列的方向”。如果机器人在校准过程中持续追头，麦克风阵列的朝向也在变化，于是同一参与者的测量角度不再属于同一参考坐标系，MAD 会增大，导致 `unstable_doa` 或错误参考角。

当前代码的直接保护是：当 `is_position_calibration_pending()` 为真时，DoA 头部跟随不应继续改变校准姿态。也就是说，**校准期间冻结定向动作，识别阶段再恢复**。

### 3.6 相对位移与绝对位移

仅根据“当前 DoA 与上一帧 DoA 的差”累加相对转角，会积累漂移；直接把 DoA 当绝对 yaw，又忽略了机器人已经转过的角度。理论上的补偿方式是把麦克风测得的相对角与当前头部/机身姿态变换合成为世界坐标中的绝对目标方向。

不过当前最终方案没有继续依赖这条复杂的动态补偿链，而是把参与者座位设为固定语义位置，使用绝对 yaw 指令。这减少了坐标漂移，也更容易解释和复现实验。

### 3.7 `fixed_lr`：保留的 DoA 简化实验模式

代码还保留了 `fixed_lr` 模式，用一个固定 DoA 分界值把一段话分到 left/right。它不需要参与者校准，而且为了端到端测试不会返回 unknown：没有有效角度时先沿用上一个 side，再回退到默认 side。

这个模式适合测试 STT → LLM → TTS 全链路，但由于它会强制给出身份，**不适合作为高可信度 speaker identification 证据**。

## 阶段四：每人一个独立麦克风（当前主方案）

### 4.1 身份从设备绑定而来

`SpeakerAssignmentController` 支持 `auto`、`manual`、`doa`、`fixed_lr` 和 `multi_mic`。在 `multi_mic` 中，每个 SoundDevice 输入配置包含：

- `device_index`：物理麦克风设备；
- `participant_id` / label：它固定属于谁；
- 运行时 source id：`mic:{device_index}:{participant_id}`。

`console.py::_record_loop_multi_sounddevice()` 为每个已绑定设备分别打开单声道输入流，把音频连同 assignment 一起交给 handler。因此身份先由配置确定，而不是从混合音频中推断。

### 4.2 不能忽略串音：当前的筛选逻辑

独立麦克风仍会同时收到房间声音，所以当前 handler 还会：

- 为每个麦克风建立噪声基线；
- 使用 VAD、RMS 和 SNR 判断其上是否存在可信语音；
- 对相近时间内的多个源组成 episode；
- 使用 transcript 相似度、RMS 比和 SNR 差识别串音副本；
- 区分一方明显占优的串音与真正的重叠讲话；
- 对低于最低 SNR 的输入不送入 STT；
- 在身份不够可靠时保留 unknown/冲突证据，而不是伪造确定结果。

因此“麦克风 A 有波形”并不直接等价于“参与者 A 正在发言”。

### 4.3 实时转头为什么不等待 STT

机器人注视 speaker 使用独立的快速通道 `_consider_live_speaker_attention()`：

```text
各麦克风实时音频块
    ↓
每源 VAD + 噪声基线 + SNR
    ↓
连续若干 speech chunks 后确认候选
    ↓
只有一个可信源 → 看向该人
多个可信源 → 只有最强源达到 dominance ratio 才看向它
否则 → 判为 ambiguous/overlap，不转头
```

这一通道不依赖 STT 文本，因此能在一句话尚未结束时动作。当前代码至少等待 5 个确认 chunk，最低 SNR 默认 6 dB；两路都活跃时，领先源强度比默认要达到 1.6 才选定目标。

### 4.4 固定座位朝向

为简化两人固定座位实验，当前动作映射写在 `handler.py::_turn_head_for_live_attention()` 中：

- Jack / Participant A / Speaker A → 机器人右侧 `+30°`；
- Mary / Participant B / Speaker B → 机器人左侧 `-30°`；
- 无明确单一目标 → 不猜测；介入面向全组时使用中心 `0°`。

动作使用 `look_at_fixed_yaw()` 发送**绝对 yaw**，不是在当前姿态上不断叠加相对角。机器人至少停留约 3 秒，然后回到中心，再重新采样当前 VAD；这样避免每个音频块都触发头部颤动。

## 当前最终数据流

```text
Jack 麦克风 ─┐
              ├─ 独立采集 → 每源 VAD/噪声/SNR → episode 与串音处理
Mary 麦克风 ─┘                         │
                                      ├─ 快速注意力：目标明确 → 固定 yaw 转头
                                      │
                                      └─ 可信语音段 → STT
                                                       ↓
                                             已绑定 participant label
                                                       ↓
                                        MeetingStateTracker.record_turn()
                                                       ↓
                                     参与平衡、介入决策、LLM/TTS
```

## 结论与当前限制

当前方案的关键原则是：**能用物理输入源确定的身份，不再用不稳定的声学推断；需要快速反应的注视，不等待完整 STT；遇到重叠或低信噪比时宁可不确定，也不错误归因。**

仍需注意：

- 麦克风插拔或 Windows 设备编号变化后必须重新确认绑定；
- 两支麦克风距离太近、增益差异过大或强串音会影响判定；
- 固定 `±30°` 只适用于当前 Jack/Mary 座位布置，不是通用人物追踪；
- `fixed_lr` 和 DoA 校准代码仍存在，但不应与当前 `multi_mic` 主方案混为一谈；
- 日志证明的是采集、判定和动作是否被排队；真实机器人是否完整听见语音、是否准确转到物理目标，仍需单独做硬件验收。

## 对应代码索引

- `src/reachy_mini_conversation_app/local/speaker_assignment.py`：speaker 模式选择与输入源兼容性；
- `src/reachy_mini_conversation_app/local/position_speaker_id.py`：DoA 校准、稳健匹配和 `fixed_lr`；
- `src/reachy_mini_conversation_app/console.py`：daemon DoA 轮询与多麦克风采集；
- `src/reachy_mini_conversation_app/local/handler.py`：speaker assignment、多麦克风串音处理、实时注意力和主 pipeline；
- `src/reachy_mini_conversation_app/config.py`：DoA、多麦克风和注意力阈值；
- `src/reachy_mini_conversation_app/moves.py`：`look_at_fixed_yaw()` 绝对 yaw 动作。
