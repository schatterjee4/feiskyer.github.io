---
title: "Gitlab故障回顾和总结"
type: page
---

## Gitlab故障回顾

1月31日，Giblab在修复一个PostgreSQL数据同步问题（DB Replication lagged too far behind）时，误将生产环境的数据删除（本来是计划删除db1上的数据，结果发现在错误的db2上操作了）。进而寻求从备份数据恢复，结果发现没有实时备份：

- LVM Snapshot每24小时备份一次，最新数据是6小时前的
- 常规备份由于pg_dump客户端版本问题失效
- Azure Disk snapshot未启用
- 数据库同步会导致webhook删除，所以webhook只能从备份中恢复
- S3 备份未生效，bucket为空
- 糟糕的备份流程，并且没有明确的文档

最后，Gitlab只能从LVM snapshot上恢复6小时前的数据。由于备份机器性能极差，并且数据拷贝极慢，整个恢复过程也比较慢（18个小时）。

## 改进措施

故障发生后，Gitlab列出了一系列的改进措施，包括

1.  [Update PS1 across all hosts to more clearly differentiate between hosts and environments (#1094)](https://gitlab.com/gitlab-com/infrastructure/issues/1094)
2.  [Prometheus monitoring for backups (#1095)](https://gitlab.com/gitlab-com/infrastructure/issues/1095)
3.  [Set PostgreSQL's max_connections to a sane value (#1096)](https://gitlab.com/gitlab-com/infrastructure/issues/1096)
4.  [Investigate Point in time recovery & continuous archiving for PostgreSQL (#1097)](https://gitlab.com/gitlab-com/infrastructure/issues/1097)
5.  [Hourly LVM snapshots of the production databases (#1098)](https://gitlab.com/gitlab-com/infrastructure/issues/1098)
6.  [Azure disk snapshots of production databases (#1099)](https://gitlab.com/gitlab-com/infrastructure/issues/1099)
7.  [Move staging to the ARM environment (#1100)](https://gitlab.com/gitlab-com/infrastructure/issues/1100)
8.  [Recover production replica(s) (#1101)](https://gitlab.com/gitlab-com/infrastructure/issues/1101)
9.  [Automated testing of recovering PostgreSQL database backups (#1102)](https://gitlab.com/gitlab-com/infrastructure/issues/1102)
10.  [Improve PostgreSQL replication documentation/runbooks (#1103)](https://gitlab.com/gitlab-com/infrastructure/issues/1103)
11.  [Investigate pgbarman for creating PostgreSQL backups (#1105)](https://gitlab.com/gitlab-com/infrastructure/issues/1105)
12.  [Investigate using WAL-E as a means of Database Backup and Realtime Replication (#494)](https://gitlab.com/gitlab-com/infrastructure/issues/494)
13.  [Build Streaming Database Restore](https://gitlab.com/gitlab-com/infrastructure/issues/1152)
14.  [Assign an owner for data durability](https://gitlab.com/gitlab-com/infrastructure/issues/1163)
15.  [Bundle pgpool-II 3.6.1 (!1251)](https://gitlab.com/gitlab-org/omnibus-gitlab/merge_requests/1251)
16.  [Connection pooling/load balancing for PostgreSQL (#259)](https://gitlab.com/gitlab-com/infrastructure/issues/259)

## 教训

- PostgreSQL配置和使用错误，参见[Dataloss at Gitlab](https://blog.2ndquadrant.com/dataloss-at-gitlab/)
- 自动化的必要性，人总是会犯错的，应该用技术而不是管理来解决问题（设计更合理的高可用系统而不是靠权限控制人肉操作）
- 备份和故障恢复系统需要定期演练，否则即便像Gitlab拥有这么多的备份系统依然会丢失数据

## 参考链接

- [Postmortem of database outage of January 31](https://about.gitlab.com/2017/02/10/postmortem-of-database-outage-of-january-31/)
- [GitLab.com Database Incident - 2017/01/31](https://docs.google.com/document/d/1GCK53YDcBWQveod9kfzW-VCxIABGiryG7_z_6jHdVik/pub)
- [GitLab.com Database Incident](https://about.gitlab.com/2017/02/01/gitlab-dot-com-database-incident/)
- [Hacker News讨论](https://news.ycombinator.com/item?id=13537052)
- [从GITLAB误删除数据库想到的](http://coolshell.cn/articles/17680.html)