# 第一节 ： PG数据库概述

## 1. 什么是 PostgreSQL :mag:

PostgreSQL 是一个功能强大的开源对象关系数据库系统，它使用并扩展了 SQL 语言，并结合了许多功能，可以安全地存储和扩展最复杂的数据工作负载。PostgreSQL 的起源可以追溯到 1986 年，是加州大学伯克利分校 [POSTGRES](https://www.postgresql.org/docs/current/history.html) 项目的一部分，并且在核心平台上已经积极开发了超过 35 年。

PostgreSQL 因其经过验证的架构、可靠性、数据完整性、强大的功能集、可扩展性以及软件背后的开源社区致力于始终如一地提供高性能和创新的解决方案而赢得了良好的声誉。PostgreSQL 可在[所有主要操作系统](https://www.postgresql.org/download/)上运行，自 2001 年以来一直符合 [ACID](https://en.wikipedia.org/wiki/ACID) 标准，并具有强大的附加组件，例如流行的 [PostGIS](https://postgis.net/) 地理空间数据库扩展器。PostgreSQL 已成为许多人和组织首选的开源关系数据库，这并不奇怪。

[开始使用](https://www.postgresql.org/docs/current/tutorial.html) PostgreSQL 从未如此简单 - 选择要构建的项目，让 PostgreSQL 安全可靠地存储您的数据。

------

PostgreSQL is a powerful, open source object-relational database system that uses and extends the SQL language combined with many features that safely store and scale the most complicated data workloads. The origins of PostgreSQL date back to 1986 as part of the [POSTGRES](https://www.postgresql.org/docs/current/history.html) project at the University of California at Berkeley and has more than 35 years of active development on the core platform.

PostgreSQL has earned a strong reputation for its proven architecture, reliability, data integrity, robust feature set, extensibility, and the dedication of the open source community behind the software to consistently deliver performant and innovative solutions. PostgreSQL runs on [all major operating systems](https://www.postgresql.org/download/), has been [ACID](https://en.wikipedia.org/wiki/ACID)-compliant since 2001, and has powerful add-ons such as the popular [PostGIS](https://postgis.net/) geospatial database extender. It is no surprise that PostgreSQL has become the open source relational database of choice for many people and organisations.

[Getting started](https://www.postgresql.org/docs/current/tutorial.html) with using PostgreSQL has never been easier - pick a project you want to build, and let PostgreSQL safely and robustly store your data.

PostgreSQL是以加州大学伯克利分校计算机系开发的[POSTGRES， 版本 4.2](http://db.cs.berkeley.edu/postgres.html)为基础的对象关系型数据库管理系统（ORDBMS）。POSTGRES 领先的许多概念在很久以后才出现在一些商业数据库系统中。

PostgreSQL是最初的伯克利代码的开源继承者。它支持大部分 SQL 标准并且提供了许多现代特性：

- 复杂查询
- 外键
- 触发器
- 可更新视图
- 事务完整性
- 多版本并发控制

同样，PostgreSQL可以用许多方法扩展，比如， 通过增加新的：

- 数据类型
- 函数
- 操作符
- 聚集函数
- 索引方法
- 过程语言

并且，因为自由宽松的许可证，任何人都可以以任何目的免费使用、修改和分发PostgreSQL， 不管是私用、商用还是学术研究目的。

## 2. PostgreSQL 简史

PostgreSQL的简史可以追溯到20世纪80年代末和90年代初，它的发展充满了创新与挑战，最终成为了广受欢迎的开源关系型数据库管理系统（RDBMS）。以下是PostgreSQL简史的一个简要概述：

### 起源与早期发展

* **项目启动**：PostgreSQL的最初构想可以追溯到1986年，当时被称为Berkeley Postgres Project，由加州大学伯克利分校的计算机科学家Michael Stonebraker及其团队启动。该项目旨在创建一个强大的、开源的关系型数据库管理系统，作为早期关系型数据库系统Ingres的继承者。
* **原型系统**：在1987年，第一个“演示性”系统已经可以使用，并在1988年的ACM-SIGMOD大会上展出。随着项目的推进，PostgreSQL逐渐从原型系统发展成为功能完善的数据库管理系统。
* **Postgres95**：在1995年，项目取得了重大进展，开发人员Andrew Yu和Jolly Chen在Postgres中添加了一个SQL（Structured Query Language，结构化查询语言）翻译程序，该版本被称为Postgres95，并在开源社区中发布。

### 开源与版本发布

* **正式更名**：1996年，Postgres95经历了较大的改动，并以PostgreSQL 6.0的版本号首次作为开源软件发布。这一决定为PostgreSQL的迅速成长和社区发展奠定了坚实的基础。
* **版本迭代**：自6.0版本以来，PostgreSQL经历了多个版本的迭代和更新。每个版本都引入了新的功能和改进，以适应不断变化的需求和技术趋势。例如，PostgreSQL 7.1加入了预写式日志功能；PostgreSQL 8.x版本开始支持Windows平台，并引入了事务保存点、表空间等功能；而PostgreSQL 9.0及后续版本则加强了复制、高可用性和性能优化等方面的能力。

### 特性与优势

* **强大功能**：PostgreSQL支持大部分SQL标准，并提供了许多其他现代特性，如复杂查询、外键、触发器、视图、事务完整性、多版本并发控制（MVCC）等。这使得PostgreSQL能够胜任各种复杂的数据处理和管理任务。
* **高度可扩展性**：PostgreSQL的架构和设计使其具有高度的可扩展性。开发人员可以通过增加新的数据类型、函数、操作符、聚集函数、索引方法等来扩展数据库的功能。此外，PostgreSQL还支持丰富的插件和扩展，以满足不同场景下的需求。
* **开源与灵活性**：作为开源软件，PostgreSQL的许可证非常灵活，任何人都可以免费使用、修改和分发它。这使得PostgreSQL在学术界、商业界和开源社区中都得到了广泛的应用和支持。

### 未来发展

随着大数据、云计算等技术的快速发展，PostgreSQL也在不断演进和适应新的技术趋势。未来，我们可以期待PostgreSQL在性能优化、可扩展性、易用性等方面取得更多的突破和进展，为更多的用户和应用场景提供更加优秀的数据库解决方案。

## 3. PG数据库的版本

PostgreSQL数据库的各个版本及其重大变化非常广泛，由于篇幅限制，我无法一一列举所有版本及其详细变化。不过，我可以概述一些关键版本及其重大变化，以帮助你了解PostgreSQL的发展历程。

### PostgreSQL 7.x 系列

* **7.0**：这是PostgreSQL以该版本号命名的第一个稳定版本，引入了预写式日志（WAL）等关键特性。
* **7.1**：进一步增强了WAL，提高了数据库的可靠性和恢复能力。

### PostgreSQL 8.x 系列

* **8.0**：增加了对Windows平台的支持，这是PostgreSQL向跨平台发展的重要一步。
* **8.1**：引入了表空间（Tablespaces）的概念，允许用户将表和索引存储在文件系统的不同位置。
* **8.2**：增加了对全文检索的支持，引入了tsearch2模块。

### PostgreSQL 9.x 系列

* **9.0**：引入了流复制（Streaming Replication）功能，为PostgreSQL提供了强大的高可用性和数据冗余能力。
* **9.1**：增强了并行查询的能力，允许在单个查询中并行处理多个数据块。
* **9.2**：增加了对JSON数据类型的原生支持，并引入了JSONB这种更高效的JSON存储格式。
* **9.3**：引入了逻辑复制（Logical Replication）功能，允许在不同数据库实例之间复制数据变更。

### PostgreSQL 10.x 系列

* **10.0**：开始使用“X.Y”的命名方式，其中X是主版本号，Y是次版本号。这个版本包含了大量的性能改进和新功能，如改进的并行查询、更快的索引构建和增强的分区表支持。

### PostgreSQL 11.x 系列

* **11.0**：继续加强并行处理能力，包括并行索引扫描和并行JOIN操作。同时，引入了通用表表达式（Common Table Expressions, CTEs）的并行化。

### PostgreSQL 12.x 系列

* **12.0**：引入了更多关于并行查询的改进，包括并行聚合和并行子查询。同时，增强了数据分区和索引功能。

### PostgreSQL 13.x 系列

* **13.0**：引入了顺序扫描加速（Incremental Sort）算法，显著提高了排序操作的性能。同时，增加了对表达式索引和存储计算列的索引的支持。

### PostgreSQL 14.x 系列

* **14.0**：增强了逻辑复制的功能，允许更细粒度的复制控制。同时，改进了备份和恢复过程，并引入了更多的安全性和性能改进。

### PostgreSQL 15.x 和 16.x 系列

* **15.0 和 16.0**：这些版本继续在性能、安全性、可扩展性和易用性方面做出改进。例如，PostgreSQL 15引入了MERGE SQL命令，而PostgreSQL 16则增强了并行查询和批量数据加载的能力。

请注意，由于PostgreSQL是一个不断发展的开源项目，因此上述信息可能不是最新的。要获取最新版本的详细信息和变化，请参考PostgreSQL的官方文档和发布说明。

## 4. 公约

下面的约定被用于命令的大纲：方括弧（`[`和`]`）表示可选的部分（在 Tcl 命令里，使用的是问号 （`?`），就像通常的 Tcl 一样）。 花括弧（`{`和`}`）和竖线（`|`）表示你必须选取一个候选。 点（`...`）表示它前面的元素可以被重复。

如果能提高清晰度，那么 SQL 命令前面会放上提示符`=>`， 而 shell 命令前面会放上提示符 `$`。不过，提示符通常不被显示。

一个*管理员*通常是一个负责安装和运行服务器的人员。 一个*用户*可以是任何使用或者需要使用PostgreSQL系统的任何部分的人员。 我们不应该对这些术语的概念理解得太狭隘；这份文档集在系统管理过程方面没有固定的假设。

PG数据库，即PostgreSQL数据库，作为一个功能强大、开源的关系型数据库管理系统，其使用和维护过程中逐渐形成了一些公认的公约或最佳实践。这些公约旨在确保数据库的性能、可维护性、安全性以及与其他系统的兼容性。以下是一些常见的PostgreSQL数据库公约：

### 1. 命名规范

* **对象命名**：包括库名、表名、列名、函数名等，应使用小写字母、下划线和数字，且首字母必须为小写。避免使用SQL保留字和生僻冷词。
* **表名**：通常使用复数名词，遵循历史惯例。
* **视图**：以`v_`作为命名前缀，物化视图使用`mv_`。
* **临时表**：以`tmp_`作为命名前缀。
* **索引**：主键索引以`_pkey`结尾，唯一索引以`_key`结尾。

### 2. 字符编码

* **字符编码**：推荐使用UTF8编码，避免使用其他字符编码，以确保数据的一致性和可移植性。

### 3. 数据类型选择

* **选择合适的数据类型**：使用正确的数据类型可以显著提高数据存储、查询、索引和计算的效率。例如，对于稳定的、取值空间较小的字段，应使用枚举类型而非整型或字符串。

### 4. 性能优化

* **索引优化**：合理创建和使用索引可以显著减少查询时间。
* **缓存优化**：通过调整数据库的缓存参数，可以提升数据访问的速度。
* **查询优化**：对SQL语句进行调整，使用更高效的查询方式。
* **硬件优化**：增加内存、使用SSD硬盘等硬件升级也可以提升数据库的性能。

### 5. 安全性

* **身份验证和访问控制**：通过多种身份验证方式（如密码、Kerberos等）确保只有授权用户能够访问数据库。
* **数据加密**：对存储的数据和传输中的数据进行加密，确保数据的安全性。
* **基于角色的访问控制**：为不同用户分配不同的权限，实现细粒度的访问控制。

### 6. 备份与恢复

* **定期备份**：使用逻辑备份或物理备份方式定期备份数据库，确保数据的安全性和可恢复性。
* **增量备份和时间点恢复**：支持增量备份和时间点恢复，以便在数据丢失或损坏时快速恢复到指定的时间点。

### 7. 插件和扩展

* **插件机制**：利用PostgreSQL的插件机制扩展其功能，如使用PostGIS插件进行地理空间数据处理。
* **兼容性**：在引入插件和扩展时，注意其与现有系统的兼容性和稳定性。

### 8. 开发与运维

* **协作**：后端、运维和DBA应密切协作，共同确保数据库的稳定运行和性能优化。
* **文档与注释**：对数据库对象、表结构、SQL语句等进行详细的文档化和注释，以便于维护和理解。

这些公约和最佳实践是PostgreSQL社区多年经验的总结，遵循这些公约可以帮助你更好地使用和维护PostgreSQL数据库。然而，请注意，随着技术的不断发展和应用场景的变化，这些公约也可能会有所更新和调整。因此，建议定期关注PostgreSQL的最新动态和最佳实践。