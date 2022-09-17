---
title: 如何进行稳定性治理与稳定性防范？前端视角
date: 2022-09-17 12:40:09
tags:
---


近2年除开发日常的业务系统外，从0到1开发了实时同屏音视频讲解工具，并接手了外呼系统，**2个都属于实时系统，对稳定性要求也更高**，也总结一下自己对稳定治理、稳定性保证的经验。

总结分为2部分，第一部分稳定性治理，第二部分稳定性保障：

1. **稳定性治理：** 如何在项目上线初期，通过稳定性治理，将业务系统稳定性大幅提高并大规模应用？
2. **稳定性保障：** 对稳定性要求较高的系统进行开发维护，有哪些技术规范与机制来保障系统稳定性？

## 简介
自己负责开发维护的2个项目均具备实时、宿主环境复杂、业务敏感度高的特点，属于对稳定性要求较高的项目，简单解释下3个特点：

- 实时：依赖websocket或第三方实时消息。
- 宿主环境复杂：依赖摄像头、麦克风、浏览器版本，浏览器权限等。
- 业务敏感度高：实时沟通中出现问题感知明显，如果出现故障直接关联销售额。
 	
在同屏项目中，从上线初期问题频发，到稳定运行并大规模使用，积累了稳定性治理的经验。
另外，因为同屏与外呼都具备实时、宿主环境复杂、业务敏感度高的特点，在日常的开发迭代中，掌握了如何通过研发规范与机制来保障系统稳定性。

## 稳定性治理
**如何在项目上线初期，通过稳定性治理，将业务系统稳定性大幅提高并大规模应用？**

我们的项目涉及云手机、webRTC厂商、实时消息厂商、实时消息服务等，页面区分B端与C端，又依赖摄像头/麦克风、浏览器权限、美颜软件等，链路长且宿主环境复杂，出现问题时定位也就更难。

项目上线初期，研发被拉进各个业务群内进行问题排查与解答，每天忙的焦头烂额，问题也五花八门，没办法分辨出真正的稳定性问题，也没精力解决真正的去解决稳定性问题，导致稳定性问题得不到进展。

我们后续梳理了稳定性治理方案，约定好问题上报流程，定期进行稳定性会议专项沟通，取得了明显的进展，将稳定性问题逐步解决，并大规模应用起来了。

### 1. 统一问题上报流程与稳定性指标
在实际业务中，用户遇到问题时，期望尽快得到解决，会通过飞书、微信等各种业务群进行询问。

有时候一个问题会在不同的群出现，有的问题没有详细的描述，只有一句话或1个截图，没办法根据极少的信息进行定位与排查；哪怕有详细数据，提供方案后是否有效，是否能解决也得不到反馈。

问题散落在个个群内，进行数据汇总的成本很高，也没办法根据数据及时找到共性规律，**群并不是高效解决稳定性问题的最优渠道，建立统一的上报流程就显得尤为重要**。

**目的：**
收集真实、客观的、可观测数据，建立有效的稳定性指标并进行治理。

**怎么做：**

1. 业务、研发达成一致，统一上报流程，所有问题都要统计，避免重复上报。
1. 有效统计：所有上报要有详细信息，包括但不限于用户信息、操作描述、截图、录屏视频等。
1. 尽可自动化统计：自动截图、将相关数据汇总，如一键提交工单功能。
1. 确立稳定性指标：将反馈次数多的问题数量确立为稳定性指标。
1. 便捷查看：通过看板、数据周报形式，能够便捷观测数据的变化。

### 2. 建立FAQ手册
并非所有问题都应该由技术去解决，可以通过培训、学习视频课程、宣导等**SOP标准化流程来解决高频问题**。

譬如“为什么提示我没权限？”、“客户没网怎么办？”、“如何打开美颜？”等诸如此类的问题；
也可以将我们无法解决的问题包含在内，譬如客户手机不支持音视频通话时，可以引导客户使用家人的手机或使用电脑；亦或是一些异常突发情况的安慰话术，如“网络或者电脑问题需要应由IT人员协助解决”等。

**目的：** 用适合的方式高效的解决业务问题，避免非技术问题透传给研发人员。

**怎么做：**

1. 建立FAQ手册：筛选可通过SOP标准化流程解决高频问题，并给出答案。
2. 对使用人员进行培训、宣导。
3. 强调问题上报前需根据FAQ提供的方案尝试自行解决。
4. 验证FAQ是否有效，观测上报问题数量是否有减少，是否需要迭代。


### 3. 建立需求池
业务需求不会因为稳定性问题而停止，一定要建立需求池列表，**将稳定性问题列入需求池排期中，让研发投入精力去排查与定位**，并最终解决。

**目的：** 遵循客观规律，技术问题需要研发人员投入精力才能解决。

**怎么做：**

1. 汇总需求列表与稳定性问题列表，并进行粗略的时间评估。
2. 与产品、业务人员确定优先级，并确立排期。
3. 按排期进行稳定性问题跟进。


### 4. 稳定性专项会议
稳定性治理初期业务人员与研发人员要密切沟通，根据问题数据、迭代流程、FAQ、需求优先级等进行确认，**研发与业务必须达成一致，防止信息不同步**，会议可以简短但不能没有。

后期稳定性治理取得较大进展后，可放缓会议频率或逐步取消。

**目的：** 小步快跑的迭代稳定性治理方案。

**怎么做：**

1. 确定稳定性会议人员、会议频率、发起人等。
2. 确定会议内容：根据上报数据、需求进展、稳定性指标，判断是否调整优先级、FAQ、上报流程等。
3. 按约定执行会议。

---

## 稳定性保障
**对稳定性要求较高的的系统进行开发维护，有哪些技术规范与机制来保障系统稳定性？**

### 技术方案：

1. 灰度策略：新需求或改动要评估影响范围，并给出新功能灰度策略，尽量避免核心功能全量发布。
1. 回测标准：根据影响范围与核心流程，梳理出回测Case，回测必须覆盖核心流程。
1. 系统日志：对核心流程增加日志记录，保证问题可回溯，有数据可查。
1. 埋点：针对正常点与异常点增加埋点统计，可根据埋点评估业务使用状况。
1. 补偿与兜底：涉及核心流程时，要对核心增加补偿机制、兜底策略，在核心流程执行失败时可重试或走兜底策略。
1. 多活机制：避免核心服务的单点依赖，如外呼的CTI厂商接入多家、RTC厂商接入多家，在依赖服务发生故障时可紧急切换，保障系统正常运行。
1. 上线计划：要避开使用高峰期上线，根据每日的使用量判断，建议在使用量末端时上线， 该事件段内有小部分流量可验证系统功能是否正常，不会导致因为回测场景不足导致的高分期故障。

### 值班机制：

1. 报错值班：前后端服务统一接入报错监控，代码异常错误时及时通知到研发人员，轮值进行报错跟踪。
1. 报警值班：指业务指标数据的报警，如登录次数、浏览量、支付量等，可环比上周或上一天数据进行监控；如果系统发生故障，关键的业务指标是非常敏感且准确的。
1. 上线值班：新功能上线前指定值班人员，提高警觉，出现问题第一时间有人响应，避免问题扩大。
1. 指标巡检：有些问题可能需要经过较长的潜伏期才能暴露，要定期多业务指标进行巡检，对比上周、上月是否有明显差异。

### 故障/Bug复盘：
当稳定性问题解决后，要及时进行归因复盘，讨论如何避免同类问题的再次发生，围绕在事前积极预防，事中快速处理 ，事后总结提高，并拆解出可落地具体事项。

例如：

1. 技术方案checklist是否需要补充。
1. 监控报警是否全面。
1. 增加单元测试是否可避免。
1. 值班机制是否要需要调整。
1. 关键指标巡检是否需要增加。

## 总结
借用一句名言：

> 过程比结果更重要，过程可追求，但结果只能由过程带来，只有掌握争取的过程，才能不断复制出好的结果。
> 培养人是过程，业绩是结果；团队值多少钱，就有多少业绩。


近两年在项目中，这些过程经验也变成了我的宝贵财富，让自己对如何进行稳定性治理、如何进行稳定性保障有了一些经验积累，希望对你也有所帮助；以上为个人的经验总结，不当之处恳请斧正。

你有没有系统稳定性相关经验，欢迎一起讨论，微信：nihaojob。
