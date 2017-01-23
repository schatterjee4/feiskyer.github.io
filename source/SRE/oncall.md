---
title: "on-call轮值"
layout: "post"
---

on-call轮值是运维团队的重要职责，目标是保障服务的可靠性和可用性。

* on-call工作平衡
    * 数量平衡：至少50%时间花在软件工程上，其余时间不超过25%用来on-call
    * 质量平衡：平均下来每个生产事件的处理（根源分析、处理、事后总结等）至少要6小时，所以每12小时轮值周期最多只能产生两个紧急事件
* 安全感：降低值班人员的压力，按照步骤解决问题，寻求外部帮助
* 避免运维压力过大：从其他团队抽调SRE，控制同一起事故的报警总数，和研发团队共同协调
* 避免运维压力不够：压力太小会导致自信心太强或自信心不够，所以要保证每季度至少2次on-call，灾难恢复演习（DiRT）等有助于技能提高和知识共享