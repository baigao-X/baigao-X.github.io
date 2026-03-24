# libwebrtc Qos<no value>

# libwebrtc QoS 体系

## 五层框架

| 层级 | 功能 | 核心组件 |
|------|------|----------|
| **拥塞控制 GCC** | 算网络能承受多大码率 | GoogCc、BandwidthEstimator |
| **发送整型 Pacing** | 按目标码率平滑发送，削峰填谷 | Pacer、IntervalBudget |
| **自适应码率 ABR** | 根据 GCC 结果调节编码器码率 | BitrateConstraint、OveruseDetector |
| **抗丢包** | 丢了就重传 / 纠错 | NACK、RTX、FEC |
| **抖动缓冲** | 抹平网络抖动，保证播放流畅 | JitterBuffer、NETeq |

**数据流**:
```
GCC 算出目标码率
        ↓
   ┌─────────┐
   │  Pacer  │  ← 削峰填谷，令牌桶平滑发送
   └─────────┘
        ↓
   实际发送 (网络)
```

---

## 1. 拥塞控制 GCC (Google Congestion Control)

算网络能承受多大码率

### 架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GoogCcNetworkController                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐ │
│  │    Probe        │    │  Acknowledged   │    │    Alr          │ │
│  │  Controller     │    │  Bitrate        │    │   Detector      │ │
│  └────────┬────────┘    └────────┬────────┘    └────────┬────────┘ │
│           │                      │                      │          │
│           └──────────┬───────────┴──────────────────────┘          │
│                      ▼                                             │
│           ┌─────────────────────┐                                  │
│           │ SendSideBandwidth   │                                  │
│           │   Estimation        │                                  │
│           │  ┌────────────────┐ │  ┌────────────────────────────┐ │
│           │  │LossBasedBweV2  │ │  │   DelayBasedBwe            │ │
│           │  │(丢包驱动)      │ │  │   (延迟驱动)                │ │
│           │  └────────────────┘ │  │  ┌────────────────────────┐ │ │
│           │  ┌────────────────┐ │  │  │   TrendlineEstimator   │ │ │
│           │  │LinkCapacity    │ │  │  │   (延迟梯度)            │ │ │
│           │  │Tracker         │ │  │  │   AimdRateControl      │ │ │
│           │  └────────────────┘ │  │  └────────────────────────┘ │ │
│           │  ┌────────────────┐ │  └────────────────────────────┘ │
│           │  │RttBasedBackoff│ │                                    │
│           │  └────────────────┘ │                                    │
│           └─────────────────────┘                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### 1.1 丢包驱动 (Loss-based)

丢 = 拥塞 → 乘性降速

| 组件 | 文件 | 说明 |
|------|------|------|
| `LossBasedBweV2` | `goog_cc/loss_based_bwe_v2.h` | 现代化丢包带宽估计 |
| `LossBasedBandwidthEstimation` | `goog_cc/loss_based_bandwidth_estimation.h` | V1 版本 |
| `AimdRateControl` | `remote_bitrate_estimator/aimd_rate_control.h` | AIMD 状态机 |

### 1.2 延迟驱动 (Delay-based)

排队时延抖动 = 拥塞

| 组件 | 文件 | 说明 |
|------|------|------|
| `DelayBasedBwe` | `goog_cc/delay_based_bwe.h` | 延迟驱动带宽估计 |
| `TrendlineEstimator` | `goog_cc/trendline_estimator.h` | 趋势线检测延迟梯度 |
| `InterArrivalDelta` | `goog_cc/inter_arrival_delta.h` | 到达间隔计算 |
| `OveruseDetector` | `remote_bitrate_estimator/overuse_detector.h` | 过度使用检测 |

### 1.3 辅助机制

| 组件 | 文件 | 说明 |
|------|------|------|
| `ProbeController` | `goog_cc/probe_controller.h` | 主动探测可用带宽 |
| `LinkCapacityTracker` | `goog_cc/send_side_bandwidth_estimation.h:38-56` | 链路容量估计 |
| `RttBasedBackoff` | `goog_cc/send_side_bandwidth_estimation.h:58-79` | 高 RTT 回退 |
| `AlrDetector` | `goog_cc/alr_detector.h` | 应用受限区域检测 |
| `NetworkStateEstimate` | `api/transport/network_types.h:273-299` | 网络状态估计 |
| `EcnMarking` | `api/transport/ecn_marking.h` | ECN 拥塞通知 |

---

## 2. 发送整型 Pacing

按目标码率平滑发送，削峰填谷

| 组件 | 文件 | 说明 |
|------|------|------|
| `Pacer` | `modules/pacing/pacer.h` | 核心 Pacer |
| `PacedSender` | `modules/pacing/paced_sender.h` | 节奏发送器 |
| `IntervalBudget` | `modules/pacing/interval_budget.h` | 令牌桶 |
| `AlrDetector` | `modules/congestion_controller/goog_cc/alr_detector.h` | ALR 预算管理 |

**原理**: Token Bucket 算法，限制发包离散度，平滑输出。

```cpp
// PacerConfig
struct PacerConfig {
  DataSize data_window;   // 令牌桶容量
  DataSize pad_window;    // 填充窗口
  TimeDelta time_window;  // 时间窗口
  DataRate data_rate() const { return data_window / time_window; }
};
```

**优先级调度**:
1. **探测包** (Probe) - 最高优先级
2. **重传包** (RTX) - 次高
3. **视频帧** (Video)
4. **填充包** (Padding) - 最低

---

## 3. 自适应码率 ABR (Adaptive Bitrate)

根据 GCC 结果调节编码器码率

**实现方式**:

| 方式 | 说明 |
|------|------|
| **码率自适应** | 调单个流的码率 |
| **Simulcast** | 发多流，接收端/中间节点选择 |
| **SVC** | 分层编码，丢增强层保基础层 |

### 3.1 视频 ABR

| 组件 | 文件 | 说明 |
|------|------|------|
| `BitrateConstraint` | `video/adaptation/bitrate_constraint.h` | 接收 GCC 输出的码率上限 |
| `OveruseFrameDetector` | `video/adaptation/overuse_frame_detector.h` | 检测编码器 CPU/GPU 过载 |
| `BalancedConstraint` | `video/adaptation/balanced_constraint.h` | 综合决策 |
| `EncodeUsageResource` | `video/adaptation/encode_usage_resource.h` | 编码资源监控 |

**决策流程**:
```
GCC 目标码率
    ↓
BitrateConstraint (取 min[GCC输出, 配置上限])
    ↓
OveruseFrameDetector (编码器能否跟上？)
    ↓
BalancedConstraint (综合: 网络 + 编码器 + 质量)
    ↓
视频编码器目标码率
```

### 3.2 Simulcast (多流并发)

同时发送多路不同分辨率/码率的流，接收端或 SFU 选择接收哪路。

| 组件 | 文件 | 说明 |
|------|------|------|
| `VideoStreamEncoder` | `video/video_stream_encoder.h` | 多流编码管理 |
| `RtpVideoSender` | `video/rtp_video_sender.h` | 多流发送 |

**示意图**:
```
发送端                    SFU/接收端
┌─────────────┐          ┌─────────────┐
│ Simulcast   │          │             │
│ Stream 1    │ ──720p──►│  选择接收   │
│ (2 Mbps)    │          │  哪路流     │
│ Stream 2    │ ──360p──►│             │
│ (1 Mbps)    │          │             │
│ Stream 3    │ ──180p──►│             │
│ (500 kbps)  │          │             │
└─────────────┘          └─────────────┘
```

### 3.3 SVC (分层编码)

单路码流分为多层：基础层 + 增强层，可丢弃增强层适配带宽。

| 组件 | 文件 | 说明 |
|------|------|------|
| `VideoStreamEncoder` | `video/video_stream_encoder.h` | SVC 编码支持 |

**示意图**:
```
┌─────────────────────────┐
│ SVC 码流                │
│ ┌───────┬───────┬─────┐ │
│ │ 基础层 │ 增强层1│增强层2│ │ ← 可丢弃增强层
│ │ 180p  │ 360p  │ 720p │ │
│ └───────┴───────┴─────┘ │
└─────────────────────────┘
```

### 3.4 音频 ABR

| 组件 | 文件 | 说明 |
|------|------|------|
| `AudioNetworkAdaptorImpl` | `audio_network_adaptor_impl.h` | 音频网络适配器 |
| `BitrateController` | `bitrate_controller.h` | 比特率控制 |
| `FrameLengthController` | `frame_length_controller.h` | 帧长度控制 |
| `DtxController` | `dtx_controller.h` | 断续传输控制 |
| `FecController` | `fec_controller_plr_based.h` | FEC 控制 |

---

## 4. 抗丢包 (Loss Protection)

丢了就重传 / 纠错

### 4.1 ARQ / NACK / RTX (重传)

| 组件 | 文件 | 说明 |
|------|------|------|
| `NackModule` | `rtp_rtcp/source/nack_module.h` | NACK 请求生成 |
| `RtpVideoSender` | `video/rtp_video_sender.h` | RTX 重传处理 |
| `NackSender` | `rtp_rtcp/source/nack_sender.h` | NACK 发送 |

**流程**:
```
接收端检测丢包
    ↓
发送 NACK
    ↓
发送端重传 (RTX)
```

### 4.2 FEC (前向纠错)

| 组件 | 文件 | 说明 |
|------|------|------|
| `ForwardErrorCorrection` | `forward_error_correction/fec/fec.h` | FEC 接口 |
| `UlpfecGenerator` | `forward_error_correction/fec/ulpfec_generator.h` | ULPFEC 生成器 |
| `FlexfecHeaderReaderWriter` | `forward_error_correction/fec/flexfec_header_reader_writer.h` | FlexFEC |

**FEC 类型**:

| 类型 | 说明 |
|------|------|
| **ULPFEC** | XOR 异或校验，简单但纠错能力有限 |
| **FlexFEC** | 更灵活，支持多流联合编码 |

### 4.3 RED (冗余编码)

发送原始数据的冗余副本，接收端可丢弃冗余包。

| 组件 | 文件 | 说明 |
|------|------|------|
| `RedPayloadSplitter` | `modules/rtp_rtcp/source/red_payload_splitter.h` | RED 打包/解包 |
| `UlpfecGenerator` | `forward_error_correction/fec/ulpfec_generator.h` | 包含 RED 生成 |

**与 FEC 区别**:
- **FEC**: 发送修复丢包的冗余数据 (XOR)
- **RED**: 直接复制发送原始数据副本，更可靠但带宽开销大

### 4.5 帧类型优先级

| 组件 | 文件 | 说明 |
|------|------|------|
| `VideoStreamEncoder` | `video/video_stream_encoder.h` | 帧调度 |

**策略**:
- **I 帧**: 最高优先级，必须可靠送达
- **P 帧**: 中等优先级
- **B 帧**: 可丢弃

---

## 5. 抖动缓冲 (Jitter Buffer)

抹平网络抖动，保证播放流畅

### 5.1 视频 Jitter Buffer

| 组件 | 文件 | 说明 |
|------|------|------|
| `JitterEstimator` | `video_coding/timing/jitter_estimator.h` | Kalman 滤波抖动估计 |
| `FrameBuffer2` | `video_coding/frame_buffer2.h` | 帧缓冲 (新版) |
| `Timing` | `video_coding/timing/timing.h` | 播放时间计算 |
| `VideoReceiver` | `video_coding/video_receiver.h` | 视频接收器 |

**JitterEstimator 原理**:
```cpp
class JitterEstimator {
  FrameDelayVariationKalmanFilter kalman_filter_;
  void UpdateEstimate(TimeDelta frame_delay, DataSize frame_size);
  TimeDelta GetJitterEstimate(double rtt_multiplier, ...);
};
```

### 5.2 音频 NETeq (Network Equity)

| 组件 | 文件 | 说明 |
|------|------|------|
| `AcmReceiver` | `audio_coding/acm2/acm_receiver.h` | NETeq 核心 |

**功能**:

| 功能 | 说明 |
|------|------|
| **抖动缓冲** | 平滑网络抖动 |
| **PLC** | 丢包补偿，预测生成丢失音频 |
| **VAD/DTX** | 语音活动检测，断续传输 |
| **加速/减速** | 时间尺度变换，调速不变调 |

---

## 文件索引

### 拥塞控制 GCC
| 文件 |
|------|
| [goog_cc_network_control.h](../src/modules/congestion_controller/goog_cc/goog_cc_network_control.h) |
| [send_side_bandwidth_estimation.h](../src/modules/congestion_controller/goog_cc/send_side_bandwidth_estimation.h) |
| [delay_based_bwe.h](../src/modules/congestion_controller/goog_cc/delay_based_bwe.h) |
| [loss_based_bwe_v2.h](../src/modules/congestion_controller/goog_cc/loss_based_bwe_v2.h) |
| [trendline_estimator.h](../src/modules/congestion_controller/goog_cc/trendline_estimator.h) |
| [aimd_rate_control.h](../src/modules/remote_bitrate_estimator/aimd_rate_control.h) |
| [probe_controller.h](../src/modules/congestion_controller/goog_cc/probe_controller.h) |
| [alr_detector.h](../src/modules/congestion_controller/goog_cc/alr_detector.h) |
| [ecn_marking.h](../src/api/transport/ecn_marking.h) |

### 自适应码率 ABR
| 文件 |
|------|
| [video/adaptation/bitrate_constraint.h](../src/video/adaptation/bitrate_constraint.h) |
| [video/adaptation/overuse_frame_detector.h](../src/video/adaptation/overuse_frame_detector.h) |
| [video/adaptation/balanced_constraint.h](../src/video/adaptation/balanced_constraint.h) |
| [video/video_stream_encoder.h](../src/video/video_stream_encoder.h) |
| [video/rtp_video_sender.h](../src/video/rtp_video_sender.h) |
| [audio_network_adaptor/audio_network_adaptor_impl.h](../src/modules/audio_coding/audio_network_adaptor/audio_network_adaptor_impl.h) |

### 发送整型 Pacing
| 文件 |
|------|
| [modules/pacing/pacer.h](../src/modules/pacing/pacer.h) |
| [modules/pacing/paced_sender.h](../src/modules/pacing/paced_sender.h) |
| [modules/pacing/interval_budget.h](../src/modules/pacing/interval_budget.h) |

### 抗丢包
| 文件 |
|------|
| [rtp_rtcp/source/nack_module.h](../src/modules/rtp_rtcp/source/nack_module.h) |
| [video/rtp_video_sender.h](../src/video/rtp_video_sender.h) |
| [forward_error_correction/fec/fec.h](../src/modules/forward_error_correction/fec/fec.h) |
| [forward_error_correction/fec/ulpfec_generator.h](../src/modules/forward_error_correction/fec/ulpfec_generator.h) |
| [rtp_rtcp/source/red_payload_splitter.h](../src/modules/rtp_rtcp/source/red_payload_splitter.h) |

### 抖动缓冲
| 文件 |
|------|
| [video_coding/timing/jitter_estimator.h](../src/modules/video_coding/timing/jitter_estimator.h) |
| [video_coding/frame_buffer2.h](../src/modules/video_coding/frame_buffer2.h) |
| [audio_coding/acm2/acm_receiver.h](../src/modules/audio_coding/acm2/acm_receiver.h) |

---

*基于 libwebrtc M138/7204 版本分析*
