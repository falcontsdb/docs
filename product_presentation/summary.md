# 海东青概述

海东青时序数据库是一款**高性能**的支持**跨平台**、**国产化**、**主从高可用**、**SQL**的时序数据库。它专门针对大量时序数据的场景实现列式存储物理引擎，提供高压缩比的低成本存储、高效的数据写入和查询服务。
  
海东青可广泛应用于物联网、车联网、工业互联网、金融、能源以及智能化IT运维等领域，能够支撑对工业设备、汽车、终端应用及云服务等数据的实时监控、风险告警、统计分析等功能需求。

海东青具备以下主要优势：
* **跨平台**：海东青只包含一个二进制文件，无需任何第三方依赖，支持跨平台部署（如Linux、Windows、Mac OS等），且同时支持私有化部署和k8s云平台部署。
* **国产化**：海东青入选信创图谱，支持国产操作系统（如麒麟、华为欧拉等）和国产硬件（如龙芯CPU、华为鲲鹏等），并取得了华为鲲鹏认证。因此海东青可作为国产化进程中的时序数据库选型。
* **开发友好**：为了提高开发者业务开发效率以及提供简洁的数据查询逻辑表达，海东青支持MySQL SQL语法和InfluxDB 语法，并兼容**InfluxDB 1.x**协议和**MySQL**协议，开发者可直接使用InfluxDB 1.x SDK或MySQL SDK两种协议操作海东青，无论哪种协议均使用同一套MySQL SQL语法（当然，运维管理也可直接使用InfluxDB shell和MySQL shell操作海东青）。
* **生态丰富**：
 	- 一些支持InfluxDB、MySQL的第三方生态工具也支持海东青。
 	- 海东青支持Datax数据迁移，用户可以在其他数据库的数据和海东青之间进行互相迁移。
 	- Grafana可直接可视化展示海东青数据。
 	- 支持Telgraf写入数据到海东青。海东青支持Prometheus协议远程写入和读取，即海东青可以作为Prometheus的底层数据存储，以及支持Prometheus的方案也可以使用海东青。同时，海东青内置暴露Prometheus协议的监控接口，运维可直接使用Prometheus拉取海东青的监控数据，便于知悉海东青的运行情况和排查运行问题。

* **主从高可用**：海东青支持主从架构，可将数据存储在多个海东青节点，从而提高数据的可靠性以避免由于单节点故障造成数据丢失。同时海东青支持主节点故障之后自动选择一个从节点提升为主节点继续提供写数据服务，从而提高海东青数据库的可用性，进而增加业务服务的可用性。

除以上介绍的主要优势之外，海东青还具备很多特别的亮点功能，您可以在[产品特性](./product_features.md)和[高级特性](../advanced_features/advanced_features_overview.md)章节中进行了解。

欢迎您垂询官网进行沟通了解，若需试用体验海东青，请从试用版下载地址下载海东青。

官网：https://www.rockontrol.com/hdqwlwsjk.jhtml

免费版下载地址：https://github.com/falcontsdb/release/releases



