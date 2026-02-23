---
title: "12306系统设计"
date: 2026-02-10T09:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags:
- go
- 位图
---

# 12306 购票系统设计详解

> 本文档详细介绍类似 12306 的火车票购票系统的核心设计，包括业务流程、数据模型、位图算法、选座算法等关键技术点。

---

## 目录

1. [业务流程概述](#一业务流程概述)
2. [系统架构设计](#二系统架构设计)
3. [数据模型设计](#三数据模型设计)
   - [车次编号规则](#33-车次编号规则)
4. [位图算法详解](#四位图算法详解)
   - [座位位图生命周期管理](#47-座位位图生命周期管理)
5. [选座算法详解](#五选座算法详解)
6. [高并发设计](#六高并发设计)
7. [完整代码实现](#七完整代码实现)

---

## 一、业务流程概述

### 1.1 用户购票流程

```
用户注册/登录 → 查询车次 → 选择车次/席别 → 选座（可选）→ 提交订单 → 支付 → 出票
                                                    ↓
                                              库存不足时
                                                    ↓
                                              候补排队 → 候补兑现 → 支付 → 出票
```

### 1.2 核心业务步骤

| 步骤 | 说明 | 关键技术点 |
|-----|------|-----------|
| 注册登录 | 用户实名认证，添加常用乘车人 | 身份证验证、手机验证 |
| 查询车次 | 输入起终点和日期，返回车次列表 | 多车次查询、余票统计 |
| 选择席别 | 商务座、一等座、二等座等 | 票价计算 |
| 选座 | 靠窗、靠过道、连座等偏好 | 选座算法 |
| 提交订单 | 锁定座位，生成待支付订单 | 分布式锁、库存扣减 |
| 支付 | 30分钟内完成支付 | 支付对接、超时处理 |
| 出票 | 生成电子客票 | 异步处理 |

---

## 二、系统架构设计

### 2.1 技术栈选型

```
后端: Go (Gin框架)
前端: Vue 3 / React
数据库: MySQL 8.0+
缓存: Redis Cluster
消息队列: RabbitMQ / Kafka
搜索: Elasticsearch (可选)
```

### 2.2 分层架构

```
┌─────────────────────────────────────────────────────────────┐
│                        用户请求                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  网关层 (Nginx/Kong)                                         │
│  - 负载均衡                                                   │
│  - 限流 (令牌桶/滑动窗口)                                      │
│  - 认证鉴权                                                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  应用层 (Go微服务)                                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│  │ 用户服务  │ │ 车次服务  │ │ 订单服务  │ │ 支付服务  │       │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  缓存层 (Redis Cluster)                                      │
│  - 余票缓存                                                   │
│  - 座位状态位图                                               │
│  - 分布式锁                                                   │
│  - 候补队列                                                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  消息队列 (RabbitMQ/Kafka)                                   │
│  - 订单异步处理                                               │
│  - 库存同步                                                   │
│  - 通知推送                                                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  数据层 (MySQL主从 + 分库分表)                                 │
│  - 用户数据                                                   │
│  - 车次数据                                                   │
│  - 订单数据                                                   │
│  - 座位状态                                                   │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 高并发请求处理流程

```
                    请求量
用户请求 ──────────► 100万/秒
    │
    ▼
┌─────────────┐
│  限流层     │     令牌桶算法，限制每用户请求频率
└─────────────┘
    │               10万/秒 (过滤90%恶意/重复请求)
    ▼
┌─────────────┐
│  缓存层     │     Redis判断是否有余票，无票直接返回
└─────────────┘
    │               1万/秒 (无票请求快速失败)
    ▼
┌─────────────┐
│  队列层     │     进入消息队列排队处理
└─────────────┘
    │               1000/秒 (削峰填谷)
    ▼
┌─────────────┐
│  数据库层   │     实际扣减库存，生成订单
└─────────────┘
```

---

## 三、数据模型设计

### 3.1 核心表结构

---

#### 3.1.1 用户表 (users)

**表说明**：存储系统注册用户的基本信息，是系统的核心账户表。用户通过手机号注册，完成实名认证后方可购票。

```sql
CREATE TABLE users (
    id              BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT '用户唯一标识，自增主键',
    phone           VARCHAR(11) NOT NULL UNIQUE COMMENT '手机号码，用于登录和接收通知，11位中国大陆手机号',
    password_hash   VARCHAR(255) NOT NULL COMMENT '密码哈希值，使用bcrypt等算法加密存储，禁止明文',
    real_name       VARCHAR(50) DEFAULT NULL COMMENT '真实姓名，实名认证后填写，用于购票校验',
    id_card         VARCHAR(18) DEFAULT NULL COMMENT '身份证号码，18位，实名认证必填，用于购票和进站验证',
    id_card_verified TINYINT UNSIGNED DEFAULT 0 COMMENT '身份证是否已验证：0-未验证 1-已验证（调用公安接口验证）',
    email           VARCHAR(100) DEFAULT NULL COMMENT '电子邮箱，可选，用于接收行程通知',
    avatar_url      VARCHAR(255) DEFAULT NULL COMMENT '用户头像URL，可选',
    status          TINYINT UNSIGNED DEFAULT 1 COMMENT '账户状态：0-禁用 1-正常 2-冻结（异常行为）',
    last_login_at   TIMESTAMP NULL COMMENT '最后登录时间，用于安全审计',
    last_login_ip   VARCHAR(45) DEFAULT NULL COMMENT '最后登录IP，支持IPv6格式',
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '账户创建时间',
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最后更新时间',

    INDEX idx_phone (phone),
    INDEX idx_id_card (id_card),
    INDEX idx_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='用户账户表';
```

**字段详细说明**：

| 字段名 | 类型 | 必填 | 说明 | 业务规则 |
|-------|------|-----|------|---------|
| id | BIGINT UNSIGNED | 是 | 主键 | 自增，全局唯一 |
| phone | VARCHAR(11) | 是 | 手机号 | 唯一索引，格式校验：1[3-9]\d{9} |
| password_hash | VARCHAR(255) | 是 | 密码哈希 | 使用bcrypt加密，cost>=10 |
| real_name | VARCHAR(50) | 否 | 真实姓名 | 实名认证后必填，2-20个汉字 |
| id_card | VARCHAR(18) | 否 | 身份证号 | 实名认证后必填，需校验校验位 |
| id_card_verified | TINYINT | 否 | 实名状态 | 调用公安部接口验证真实性 |
| status | TINYINT | 是 | 账户状态 | 异常登录/刷票行为会被冻结 |

---

#### 3.1.2 乘车人表 (passengers)

**表说明**：存储用户添加的常用乘车人信息。一个用户可以添加多个乘车人（如家人、同事），购票时选择乘车人即可自动填充信息。12306限制每个用户最多添加15个常用联系人。

```sql
CREATE TABLE passengers (
    id              BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT '乘车人唯一标识',
    user_id         BIGINT UNSIGNED NOT NULL COMMENT '所属用户ID，外键关联users表',
    name            VARCHAR(50) NOT NULL COMMENT '乘车人姓名，必须与证件姓名一致',
    id_type         TINYINT UNSIGNED DEFAULT 1 COMMENT '证件类型：1-二代身份证 2-港澳通行证 3-台湾通行证 4-护照',
    id_number       VARCHAR(30) NOT NULL COMMENT '证件号码，根据证件类型不同长度不同',
    phone           VARCHAR(11) DEFAULT NULL COMMENT '乘车人手机号，用于接收行程通知短信',
    passenger_type  TINYINT UNSIGNED DEFAULT 1 COMMENT '乘客类型：1-成人 2-儿童 3-学生 4-残疾军人',
    is_verified     TINYINT UNSIGNED DEFAULT 0 COMMENT '是否已通过12306核验：0-待核验 1-已通过 2-未通过',
    is_default      TINYINT UNSIGNED DEFAULT 0 COMMENT '是否为默认乘车人：0-否 1-是，每用户仅一个',
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '添加时间',
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',

    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    UNIQUE KEY uk_user_idnumber (user_id, id_type, id_number) COMMENT '同一用户同类型证件号唯一',
    INDEX idx_user_id (user_id),
    INDEX idx_id_number (id_number)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='常用乘车人表';
```

**字段详细说明**：

| 字段名 | 类型 | 必填 | 说明 | 业务规则 |
|-------|------|-----|------|---------|
| id_type | TINYINT | 是 | 证件类型 | 1:身份证 2:港澳通行证 3:台湾通行证 4:护照 |
| id_number | VARCHAR(30) | 是 | 证件号码 | 身份证18位，护照9位，需格式校验 |
| passenger_type | TINYINT | 是 | 乘客类型 | 儿童票(1.2-1.5m)半价，学生票75折(限硬座/二等座) |
| is_verified | TINYINT | 是 | 核验状态 | 未通过核验的乘车人无法购票 |

**乘客类型与票价关系**：

| 类型 | 说明 | 票价规则 |
|-----|------|---------|
| 成人 | 普通成年人 | 全价 |
| 儿童 | 身高1.2m-1.5m | 半价，无座位 |
| 学生 | 持学生证 | 硬座/二等座75折，限每年4次 |
| 残疾军人 | 持残疾军人证 | 半价 |

---

#### 3.1.3 车站表 (stations)

**表说明**：存储全国铁路车站信息。包含车站名称、所属城市、电报码等。电报码是铁路系统内部使用的3位字母代码，用于快速标识车站。

```sql
CREATE TABLE stations (
    id              BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT '车站唯一标识',
    name            VARCHAR(50) NOT NULL COMMENT '车站名称，如"北京南"、"上海虹桥"',
    city            VARCHAR(50) NOT NULL COMMENT '所属城市，如"北京"、"上海"',
    province        VARCHAR(50) DEFAULT NULL COMMENT '所属省份，如"北京市"、"上海市"',
    pinyin          VARCHAR(100) NOT NULL COMMENT '拼音全拼，用于搜索，如"beijingnan"',
    pinyin_abbr     VARCHAR(20) NOT NULL COMMENT '拼音首字母缩写，如"bjn"',
    code            VARCHAR(10) NOT NULL COMMENT '电报码，铁路系统唯一标识，如"VNP"(北京南)',
    station_type    TINYINT UNSIGNED DEFAULT 1 COMMENT '车站等级：1-特等站 2-一等站 3-二等站 4-三等站',
    longitude       DECIMAL(10, 6) DEFAULT NULL COMMENT '经度，用于地图定位',
    latitude        DECIMAL(10, 6) DEFAULT NULL COMMENT '纬度，用于地图定位',
    is_enabled      TINYINT UNSIGNED DEFAULT 1 COMMENT '是否启用：0-停用 1-启用',
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',

    UNIQUE KEY uk_code (code),
    UNIQUE KEY uk_name (name),
    INDEX idx_city (city),
    INDEX idx_pinyin (pinyin),
    INDEX idx_pinyin_abbr (pinyin_abbr)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='铁路车站表';
```

**字段详细说明**：

| 字段名 | 类型 | 必填 | 说明 | 示例 |
|-------|------|-----|------|------|
| name | VARCHAR(50) | 是 | 车站全名 | 北京南、上海虹桥、广州南 |
| city | VARCHAR(50) | 是 | 所属城市 | 北京、上海、广州 |
| pinyin | VARCHAR(100) | 是 | 拼音全拼 | beijingnan、shanghaihongqiao |
| pinyin_abbr | VARCHAR(20) | 是 | 拼音缩写 | bjn、shhq |
| code | VARCHAR(10) | 是 | 电报码 | VNP(北京南)、AOH(上海虹桥) |

**常见电报码示例**：

| 车站名 | 电报码 | 说明 |
|-------|-------|------|
| 北京 | BJP | Beijing Passenger |
| 北京南 | VNP | - |
| 上海 | SHH | Shanghai |
| 上海虹桥 | AOH | - |
| 广州南 | IZQ | - |

---

#### 3.1.4 列车表 (trains)

**表说明**：存储列车（车次）基本信息。注意：上行和下行是两个独立的车次记录（如D123和D124）。

```sql
CREATE TABLE trains (
    id              BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT '列车唯一标识',
    train_no        VARCHAR(10) NOT NULL COMMENT '车次号，如G1、D123、K567',
    train_type      VARCHAR(20) NOT NULL COMMENT '列车类型：G-高铁 D-动车 C-城际 Z-直达 T-特快 K-快速',
    train_code      VARCHAR(20) DEFAULT NULL COMMENT '列车编组号，如CRH380A-001，标识物理列车',
    total_stops     INT UNSIGNED NOT NULL COMMENT '总站数（含起终点）',
    total_distance  INT UNSIGNED DEFAULT NULL COMMENT '全程公里数，用于计算票价',
    start_station_id BIGINT UNSIGNED NOT NULL COMMENT '始发站ID',
    end_station_id  BIGINT UNSIGNED NOT NULL COMMENT '终点站ID',
    start_time      TIME NOT NULL COMMENT '始发时间',
    end_time        TIME NOT NULL COMMENT '终到时间',
    duration_minutes INT UNSIGNED NOT NULL COMMENT '全程运行时间（分钟）',
    is_cross_day    TINYINT UNSIGNED DEFAULT 0 COMMENT '是否跨日运行：0-否 1-是',
    run_days        VARCHAR(20) DEFAULT '1111111' COMMENT '运行日期，7位字符表示周一到周日，1-运行 0-不运行',
    sale_start_days INT UNSIGNED DEFAULT 15 COMMENT '提前售票天数，默认15天',
    status          TINYINT UNSIGNED DEFAULT 1 COMMENT '状态：0-停运 1-正常 2-临时停运',
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',

    UNIQUE KEY uk_train_no (train_no),
    FOREIGN KEY (start_station_id) REFERENCES stations(id),
    FOREIGN KEY (end_station_id) REFERENCES stations(id),
    INDEX idx_type (train_type),
    INDEX idx_start_station (start_station_id),
    INDEX idx_end_station (end_station_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='列车/车次表';
```

**字段详细说明**：

| 字段名 | 类型 | 必填 | 说明 | 示例 |
|-------|------|-----|------|------|
| train_no | VARCHAR(10) | 是 | 车次号 | G1, D123, K567 |
| train_type | VARCHAR(20) | 是 | 列车类型 | G/D/C/Z/T/K |
| total_stops | INT | 是 | 总站数 | 5（表示有4个区间段） |
| run_days | VARCHAR(20) | 否 | 运行日 | "1111111"=每天, "1111100"=周一至周五 |
| is_cross_day | TINYINT | 否 | 是否跨日 | 晚上发车次日到达的车次 |

**列车类型与速度对应**：

| 类型代码 | 名称 | 最高速度 | 说明 |
|---------|------|---------|------|
| G | 高速动车组 | 350 km/h | 时速300公里以上 |
| D | 普通动车组 | 250 km/h | 时速200-250公里 |
| C | 城际列车 | 200 km/h | 城市群内短途 |
| Z | 直达特快 | 160 km/h | 中途不停或少停 |
| T | 特快 | 140 km/h | 停站较少 |
| K | 快速 | 120 km/h | 普通快车 |
| 纯数字 | 普快 | 100 km/h | 普通旅客列车 |

---

#### 3.1.5 列车经停表 (train_stops)

**表说明**：存储每趟列车的经停站点信息，包括到站时间、发车时间、停留时间等。这是位图算法的基础数据，站序(stop_order)决定了区间掩码的计算。

```sql
CREATE TABLE train_stops (
    id              BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT '经停记录唯一标识',
    train_id        BIGINT UNSIGNED NOT NULL COMMENT '所属列车ID，外键关联trains表',
    station_id      BIGINT UNSIGNED NOT NULL COMMENT '车站ID，外键关联stations表',
    stop_order      INT UNSIGNED NOT NULL COMMENT '站序，从0开始。0=始发站，用于计算区间掩码',
    arrive_time     TIME DEFAULT NULL COMMENT '到达时间，始发站为NULL',
    depart_time     TIME DEFAULT NULL COMMENT '出发时间，终点站为NULL',
    stop_minutes    INT UNSIGNED DEFAULT 0 COMMENT '停留时间（分钟），过路站一般2-5分钟',
    distance_km     INT UNSIGNED DEFAULT 0 COMMENT '距始发站公里数，用于计算区间票价',
    is_origin       TINYINT UNSIGNED DEFAULT 0 COMMENT '是否始发站：0-否 1-是',
    is_terminal     TINYINT UNSIGNED DEFAULT 0 COMMENT '是否终点站：0-否 1-是',
    day_offset      TINYINT UNSIGNED DEFAULT 0 COMMENT '相对发车日的日期偏移，0=当天，1=次日',
    platform        VARCHAR(10) DEFAULT NULL COMMENT '站台号，如"3站台"',
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',

    FOREIGN KEY (train_id) REFERENCES trains(id) ON DELETE CASCADE,
    FOREIGN KEY (station_id) REFERENCES stations(id),
    UNIQUE KEY uk_train_stop (train_id, stop_order) COMMENT '同一车次站序唯一',
    UNIQUE KEY uk_train_station (train_id, station_id) COMMENT '同一车次同一站唯一',
    INDEX idx_station (station_id),
    INDEX idx_train_order (train_id, stop_order)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='列车经停站表';
```

**字段详细说明**：

| 字段名 | 类型 | 必填 | 说明 | 业务规则 |
|-------|------|-----|------|---------|
| stop_order | INT | 是 | 站序 | 从0开始，用于位图掩码计算 |
| arrive_time | TIME | 否 | 到站时间 | 始发站为NULL |
| depart_time | TIME | 否 | 发车时间 | 终点站为NULL |
| day_offset | TINYINT | 否 | 日期偏移 | 跨日车次使用，0=当天，1=次日 |
| distance_km | INT | 否 | 里程数 | 用于计算区间票价 |

**站序与位图的关系**（重要）：

```
G1234 经停站:
┌────────┬─────────┬────────┬────────┬───────┬────────────────┐
│ 站序   │ 车站    │ 到达   │ 出发   │ 停留  │ 对应区间段      │
├────────┼─────────┼────────┼────────┼───────┼────────────────┤
│ 0      │ 北京南  │ -      │ 08:00  │ -     │ 区间0起点      │
│ 1      │ 济南西  │ 10:30  │ 10:32  │ 2分   │ 区间0终点/1起点│
│ 2      │ 南京南  │ 13:00  │ 13:03  │ 3分   │ 区间1终点/2起点│
│ 3      │ 上海虹桥│ 14:30  │ -      │ -     │ 区间2终点      │
└────────┴─────────┴────────┴────────┴───────┴────────────────┘

区间段: 0(北京→济南), 1(济南→南京), 2(南京→上海)
位图:   bit0          bit1          bit2
```

---

#### 3.1.6 车厢表 (carriages)

**表说明**：存储每趟列车的车厢信息。不同车厢可能有不同的席别（商务座、一等座、二等座等）和座位数量。

```sql
CREATE TABLE carriages (
    id              BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT '车厢唯一标识',
    train_id        BIGINT UNSIGNED NOT NULL COMMENT '所属列车ID',
    carriage_no     INT UNSIGNED NOT NULL COMMENT '车厢号，从1开始',
    seat_type       VARCHAR(20) NOT NULL COMMENT '席别：商务座/一等座/二等座/软卧/硬卧/硬座/无座',
    seat_count      INT UNSIGNED NOT NULL COMMENT '座位/铺位总数',
    row_count       INT UNSIGNED NOT NULL COMMENT '排数',
    layout          VARCHAR(20) NOT NULL COMMENT '座位布局，如"3+2"(二等座)、"2+2"(一等座)、"2+1"(商务座)',
    has_power       TINYINT UNSIGNED DEFAULT 1 COMMENT '是否有充电插座：0-无 1-有',
    is_quiet        TINYINT UNSIGNED DEFAULT 0 COMMENT '是否静音车厢：0-否 1-是',
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',

    FOREIGN KEY (train_id) REFERENCES trains(id) ON DELETE CASCADE,
    UNIQUE KEY uk_train_carriage (train_id, carriage_no),
    INDEX idx_train_type (train_id, seat_type)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='车厢表';
```

**字段详细说明**：

| 字段名 | 类型 | 必填 | 说明 | 示例 |
|-------|------|-----|------|------|
| carriage_no | INT | 是 | 车厢号 | 1, 2, 3... |
| seat_type | VARCHAR(20) | 是 | 席别 | 商务座/一等座/二等座 |
| layout | VARCHAR(20) | 是 | 布局 | "3+2"=每排5座, "2+2"=每排4座 |
| is_quiet | TINYINT | 否 | 静音车厢 | 高铁3号车厢通常为静音车厢 |

**典型高铁车厢配置**（8节编组）：

| 车厢号 | 席别 | 座位数 | 布局 | 说明 |
|-------|------|-------|------|------|
| 1 | 商务座 | 24 | 2+1 | 3排×3座/排 |
| 2 | 一等座 | 52 | 2+2 | 13排×4座/排 |
| 3 | 二等座(静音) | 90 | 3+2 | 18排×5座/排 |
| 4-7 | 二等座 | 90 | 3+2 | 18排×5座/排 |
| 8 | 二等座 | 77 | 3+2 | 含残疾人座位 |

---

#### 3.1.7 座位表 (seats)

**表说明**：存储每节车厢的具体座位信息，包括座位号、排号、列号、位置类型等。这是库存管理的最小单位。

```sql
CREATE TABLE seats (
    id              BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT '座位唯一标识',
    carriage_id     BIGINT UNSIGNED NOT NULL COMMENT '所属车厢ID',
    seat_no         VARCHAR(10) NOT NULL COMMENT '座位号，格式：排号+列号，如"01A"、"15F"',
    row_num         INT UNSIGNED NOT NULL COMMENT '排号，从1开始',
    col_code        CHAR(1) NOT NULL COMMENT '列号：A/B/C/D/F（注意没有E）',
    position        VARCHAR(10) NOT NULL COMMENT '位置类型：window-靠窗 aisle-靠过道 middle-中间',
    is_available    TINYINT UNSIGNED DEFAULT 1 COMMENT '座位是否可用：0-不可用(设备故障等) 1-可用',
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',

    FOREIGN KEY (carriage_id) REFERENCES carriages(id) ON DELETE CASCADE,
    UNIQUE KEY uk_carriage_seat (carriage_id, seat_no),
    INDEX idx_carriage (carriage_id),
    INDEX idx_position (position)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='座位表';
```

**字段详细说明**：

| 字段名 | 类型 | 必填 | 说明 | 示例 |
|-------|------|-----|------|------|
| seat_no | VARCHAR(10) | 是 | 座位号 | 01A, 05C, 18F |
| row_num | INT | 是 | 排号 | 1-18 (二等座) |
| col_code | CHAR(1) | 是 | 列号 | A/B/C/D/F (无E) |
| position | VARCHAR(10) | 是 | 位置 | window/aisle/middle |

**座位列号与位置对应关系**：

| 席别 | 布局 | A | B | C | D | F |
|-----|------|---|---|---|---|---|
| 二等座 | 3+2 | 靠窗 | 中间 | 靠过道 | 靠过道 | 靠窗 |
| 一等座 | 2+2 | 靠窗 | - | 靠过道 | 靠过道 | 靠窗 |
| 商务座 | 2+1 | 靠窗 | - | 靠过道 | - | 靠窗(单独) |

**座位布局示意**（二等座）：

```
        窗                    过道                    窗
        │    A    B    C    │    D    F    │
        ├────────────────────────────────────┤
   01   │   01A  01B  01C   │   01D  01F   │
   02   │   02A  02B  02C   │   02D  02F   │
   ...  │   ...  ...  ...   │   ...  ...   │
   18   │   18A  18B  18C   │   18D  18F   │
        └────────────────────────────────────┘
```

---

#### 3.1.8 每日座位状态表 (daily_seat_status) ⭐核心表

**表说明**：这是购票系统最核心的表，记录每个座位在每个运营日期的售卖状态。使用位图(bitmap)存储区间占用情况，支持同一座位在不同区间卖给不同乘客。

```sql
CREATE TABLE daily_seat_status (
    id              BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT '记录唯一标识',
    seat_id         BIGINT UNSIGNED NOT NULL COMMENT '座位ID，关联seats表',
    train_id        BIGINT UNSIGNED NOT NULL COMMENT '列车ID，冗余字段便于查询',
    departure_date  DATE NOT NULL COMMENT '发车日期，如2024-01-15',
    status          BIGINT UNSIGNED DEFAULT 0 COMMENT '位图状态，每个bit表示一个区间是否已售：0-未售 1-已售',
    lock_version    INT UNSIGNED DEFAULT 0 COMMENT '乐观锁版本号，防止并发超卖',
    locked_until    TIMESTAMP NULL COMMENT '座位锁定截止时间，用于支付倒计时',
    locked_by       BIGINT UNSIGNED DEFAULT NULL COMMENT '锁定该座位的订单ID',
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',

    FOREIGN KEY (seat_id) REFERENCES seats(id),
    FOREIGN KEY (train_id) REFERENCES trains(id),
    UNIQUE KEY uk_seat_date (seat_id, departure_date) COMMENT '同一座位同一日期唯一',
    INDEX idx_train_date (train_id, departure_date) COMMENT '按车次和日期查询',
    INDEX idx_train_date_status (train_id, departure_date, status) COMMENT '查询可售座位',
    INDEX idx_locked_until (locked_until) COMMENT '清理过期锁定'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='每日座位状态表（位图存储）';
```

**字段详细说明**：

| 字段名 | 类型 | 必填 | 说明 | 重要程度 |
|-------|------|-----|------|---------|
| seat_id | BIGINT | 是 | 座位ID | 与departure_date组成唯一键 |
| departure_date | DATE | 是 | 发车日期 | 按日期隔离数据 |
| status | BIGINT | 是 | **位图状态** | ⭐核心字段，每bit表示一个区间 |
| lock_version | INT | 是 | 乐观锁版本 | 防止并发更新冲突 |
| locked_until | TIMESTAMP | 否 | 锁定截止时间 | 用户下单后锁定15分钟 |

**status 字段位图解释**（重要）：

```
假设列车有4站（3个区间）：北京→济南→南京→上海

status = 0b000 (0)  → 全程可售
status = 0b001 (1)  → 区间0(北京→济南)已售
status = 0b010 (2)  → 区间1(济南→南京)已售
status = 0b011 (3)  → 区间0,1(北京→南京)已售
status = 0b101 (5)  → 区间0,2已售，区间1可售
status = 0b111 (7)  → 全程已售，不可再售

判断是否可售: (status & 区间掩码) == 0
购票更新:    status = status | 区间掩码
退票更新:    status = status &^ 区间掩码
```

**乐观锁使用示例**：

```sql
-- 购票时更新座位状态（带乐观锁）
UPDATE daily_seat_status
SET status = status | @mask,
    lock_version = lock_version + 1,
    updated_at = NOW()
WHERE seat_id = @seat_id
  AND departure_date = @date
  AND lock_version = @current_version  -- 乐观锁检查
  AND (status & @mask) = 0;            -- 确保区间未被占用

-- 如果affected_rows = 0，说明被其他人抢先购买，需要重新选座
```

---

#### 3.1.9 订单表 (orders)

**表说明**：存储用户提交的订单信息。一个订单可以包含多张票（为多个乘车人购买）。订单有30分钟支付时限，超时自动取消。

```sql
CREATE TABLE orders (
    id              BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT '订单ID',
    order_no        VARCHAR(32) NOT NULL COMMENT '订单号，格式：日期+随机数，如202401150001234567',
    user_id         BIGINT UNSIGNED NOT NULL COMMENT '下单用户ID',
    status          TINYINT UNSIGNED DEFAULT 0 COMMENT '订单状态：0-待支付 1-已支付 2-已取消 3-已退款 4-部分退款',
    total_amount    DECIMAL(10,2) NOT NULL COMMENT '订单总金额（元），所有票价之和',
    discount_amount DECIMAL(10,2) DEFAULT 0 COMMENT '优惠金额（元）',
    pay_amount      DECIMAL(10,2) NOT NULL COMMENT '实付金额（元）= 总金额 - 优惠',
    ticket_count    INT UNSIGNED NOT NULL COMMENT '购票数量',
    contact_name    VARCHAR(50) NOT NULL COMMENT '联系人姓名',
    contact_phone   VARCHAR(11) NOT NULL COMMENT '联系人手机号，用于接收通知',
    pay_type        TINYINT UNSIGNED DEFAULT NULL COMMENT '支付方式：1-支付宝 2-微信 3-银联 4-余额',
    pay_time        TIMESTAMP NULL COMMENT '支付完成时间',
    pay_trade_no    VARCHAR(64) DEFAULT NULL COMMENT '第三方支付流水号',
    expire_time     TIMESTAMP NOT NULL COMMENT '订单过期时间，超时未支付自动取消',
    cancel_reason   VARCHAR(255) DEFAULT NULL COMMENT '取消原因',
    refund_amount   DECIMAL(10,2) DEFAULT 0 COMMENT '已退款金额',
    remark          VARCHAR(255) DEFAULT NULL COMMENT '订单备注',
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '下单时间',
    updated_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',

    UNIQUE KEY uk_order_no (order_no),
    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_user_id (user_id),
    INDEX idx_status (status),
    INDEX idx_expire_time (expire_time) COMMENT '用于超时订单处理',
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='订单主表';
```

**字段详细说明**：

| 字段名 | 类型 | 必填 | 说明 | 业务规则 |
|-------|------|-----|------|---------|
| order_no | VARCHAR(32) | 是 | 订单号 | 全局唯一，格式：yyyyMMdd+10位序列 |
| status | TINYINT | 是 | 订单状态 | 状态机流转见下表 |
| expire_time | TIMESTAMP | 是 | 过期时间 | 下单后30分钟，定时任务扫描取消 |
| pay_trade_no | VARCHAR(64) | 否 | 支付流水 | 对账使用 |

**订单状态流转**：

```
              ┌──────────────────────────────────────┐
              │                                      │
              ▼                                      │
┌─────────┐  支付成功  ┌─────────┐  全部退票  ┌─────────┐
│ 待支付  │ ────────► │ 已支付  │ ────────► │ 已退款  │
│   (0)   │           │   (1)   │           │   (3)   │
└─────────┘           └─────────┘           └─────────┘
     │                     │
     │ 超时/取消            │ 部分退票
     ▼                     ▼
┌─────────┐           ┌─────────┐
│ 已取消  │           │部分退款 │
│   (2)   │           │   (4)   │
└─────────┘           └─────────┘
```

---

#### 3.1.10 订单明细表 (order_items)

**表说明**：存储订单中每张票的详细信息。一个订单可能包含多张票（为多个乘车人购买），每张票对应一条明细记录。

```sql
CREATE TABLE order_items (
    id                  BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT '明细ID',
    order_id            BIGINT UNSIGNED NOT NULL COMMENT '所属订单ID',
    order_no            VARCHAR(32) NOT NULL COMMENT '订单号，冗余便于查询',
    passenger_id        BIGINT UNSIGNED NOT NULL COMMENT '乘车人ID',
    passenger_name      VARCHAR(50) NOT NULL COMMENT '乘车人姓名，冗余便于展示',
    passenger_id_number VARCHAR(30) NOT NULL COMMENT '乘车人证件号，冗余便于展示',
    train_id            BIGINT UNSIGNED NOT NULL COMMENT '列车ID',
    train_no            VARCHAR(10) NOT NULL COMMENT '车次号，冗余便于展示',
    seat_id             BIGINT UNSIGNED NOT NULL COMMENT '座位ID',
    carriage_no         INT UNSIGNED NOT NULL COMMENT '车厢号，冗余便于展示',
    seat_no             VARCHAR(10) NOT NULL COMMENT '座位号，冗余便于展示',
    seat_type           VARCHAR(20) NOT NULL COMMENT '席别，冗余便于展示',
    departure_station_id BIGINT UNSIGNED NOT NULL COMMENT '出发站ID',
    departure_station   VARCHAR(50) NOT NULL COMMENT '出发站名，冗余便于展示',
    arrival_station_id  BIGINT UNSIGNED NOT NULL COMMENT '到达站ID',
    arrival_station     VARCHAR(50) NOT NULL COMMENT '到达站名，冗余便于展示',
    departure_date      DATE NOT NULL COMMENT '发车日期',
    departure_time      TIME NOT NULL COMMENT '发车时间',
    arrival_date        DATE NOT NULL COMMENT '到达日期（可能次日）',
    arrival_time        TIME NOT NULL COMMENT '到达时间',
    ticket_type         TINYINT UNSIGNED DEFAULT 1 COMMENT '票种：1-成人票 2-儿童票 3-学生票 4-残疾军人票',
    price               DECIMAL(10,2) NOT NULL COMMENT '票面价格（元）',
    status              TINYINT UNSIGNED DEFAULT 0 COMMENT '状态：0-待支付 1-已支付 2-已退票 3-已改签',
    check_in_status     TINYINT UNSIGNED DEFAULT 0 COMMENT '检票状态：0-未检票 1-已检票',
    refund_amount       DECIMAL(10,2) DEFAULT 0 COMMENT '退票金额',
    refund_time         TIMESTAMP NULL COMMENT '退票时间',
    segment_mask        BIGINT UNSIGNED NOT NULL COMMENT '占用的区间掩码，退票时用于恢复座位状态',
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',

    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    INDEX idx_order_id (order_id),
    INDEX idx_order_no (order_no),
    INDEX idx_passenger (passenger_id),
    INDEX idx_train_date (train_id, departure_date),
    INDEX idx_seat_date (seat_id, departure_date)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='订单明细表（车票表）';
```

**字段详细说明**：

| 字段名 | 类型 | 必填 | 说明 | 业务规则 |
|-------|------|-----|------|---------|
| segment_mask | BIGINT | 是 | 区间掩码 | ⭐重要：退票时用于恢复座位状态 |
| ticket_type | TINYINT | 是 | 票种 | 影响价格：儿童半价，学生75折 |
| check_in_status | TINYINT | 是 | 检票状态 | 已检票的票不能退 |

**冗余字段说明**：

表中有多个冗余字段（如train_no、seat_no、departure_station等），原因：
1. **避免多表JOIN**：订单查询频繁，冗余可减少JOIN提升性能
2. **历史数据一致性**：即使车站改名，历史订单仍显示原名称
3. **数据快照**：购票时刻的信息快照

---

#### 3.1.11 票价表 (ticket_prices)

**表说明**：存储每趟列车各区间各席别的票价。票价 = 基础票价 + 里程费（可选设计）。

```sql
CREATE TABLE ticket_prices (
    id                  BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT COMMENT '票价记录ID',
    train_id            BIGINT UNSIGNED NOT NULL COMMENT '列车ID',
    departure_station_id BIGINT UNSIGNED NOT NULL COMMENT '出发站ID',
    arrival_station_id  BIGINT UNSIGNED NOT NULL COMMENT '到达站ID',
    departure_order     INT UNSIGNED NOT NULL COMMENT '出发站序',
    arrival_order       INT UNSIGNED NOT NULL COMMENT '到达站序',
    seat_type           VARCHAR(20) NOT NULL COMMENT '席别',
    price               DECIMAL(10,2) NOT NULL COMMENT '票价（元）',
    distance_km         INT UNSIGNED DEFAULT 0 COMMENT '区间里程（公里）',
    effective_date      DATE NOT NULL COMMENT '生效日期',
    expire_date         DATE DEFAULT '9999-12-31' COMMENT '失效日期',
    created_at          TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',

    FOREIGN KEY (train_id) REFERENCES trains(id),
    UNIQUE KEY uk_price (train_id, departure_station_id, arrival_station_id, seat_type, effective_date),
    INDEX idx_train_seat (train_id, seat_type)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='车票票价表';
```

---

### 3.2 ER 关系图

```
users 1───* passengers
  │
  │ 1
  │
  ▼ *
orders 1───* order_items
                  │
                  │ *
                  ▼ 1
trains 1───* train_stops *───1 stations
  │
  │ 1
  ▼ *
carriages 1───* seats 1───* daily_seat_status
```

### 3.3 车次编号规则

#### 3.3.1 上行与下行

中国铁路车次编号遵循**单数下行、双数上行**的规则：

| 方向 | 编号规则 | 示例 | 说明 |
|-----|---------|------|------|
| 下行 | **单数** | D123, G1, K567 | 从北京出发，或从小站到大站 |
| 上行 | **双数** | D124, G2, K568 | 回北京，或从大站到小站 |

**示例：**
- `D123`：北京 → 南京（下行）
- `D124`：南京 → 北京（上行）

D123 和 D124 是**一对往返车次**，通常由同一组列车担当，但在系统中是两个完全独立的车次。

#### 3.3.2 系统设计影响

```
物理上: 同一列火车 (CRH380A-001)
        早上从北京开到南京 (D123)
        下午从南京开回北京 (D124)

逻辑上: 两个独立车次
        ┌─────────────────────────────┐
        │ D123                        │
        │ train_id = 1001             │
        │ 经停: 北京→济南→南京         │
        │ 座位状态: 独立的位图          │
        └─────────────────────────────┘

        ┌─────────────────────────────┐
        │ D124                        │
        │ train_id = 1002             │
        │ 经停: 南京→济南→北京         │  ← 站序相反
        │ 座位状态: 独立的位图          │
        └─────────────────────────────┘
```

**数据库存储示例：**

```sql
-- trains 表
INSERT INTO trains VALUES (1001, 'D123', '动车', 3);
INSERT INTO trains VALUES (1002, 'D124', '动车', 3);

-- train_stops 表 (D123 下行)
INSERT INTO train_stops VALUES (1001, 北京站ID, 0, NULL, '08:00');
INSERT INTO train_stops VALUES (1001, 济南站ID, 1, '10:30', '10:32');
INSERT INTO train_stops VALUES (1001, 南京站ID, 2, '13:00', NULL);

-- train_stops 表 (D124 上行，站序相反)
INSERT INTO train_stops VALUES (1002, 南京站ID, 0, NULL, '14:00');
INSERT INTO train_stops VALUES (1002, 济南站ID, 1, '16:30', '16:32');
INSERT INTO train_stops VALUES (1002, 北京站ID, 2, '19:00', NULL);
```

**位图独立计算：**

```
D123 (北京→南京):
  站序: 北京=0, 济南=1, 南京=2
  买 北京→济南 的掩码 = 0b01

D124 (南京→北京):
  站序: 南京=0, 济南=1, 北京=2   ← 注意：站序重新编排
  买 南京→济南 的掩码 = 0b01
```

#### 3.3.3 车次类型速查

| 车次前缀 | 类型 | 速度 |
|---------|------|------|
| G | 高速动车组 | 300-350 km/h |
| D | 普通动车组 | 200-250 km/h |
| C | 城际列车 | 200 km/h |
| Z | 直达特快 | 160 km/h |
| T | 特快 | 140 km/h |
| K | 快速 | 120 km/h |
| 纯数字 | 普快 | 100 km/h |

---

## 四、位图算法详解

### 4.1 为什么使用位图？

**传统方案的问题：**

假设一趟列车有 5 个站：`北京 → 济南 → 南京 → 杭州 → 上海`

如果为每个"起终点对"单独记录库存：
- 北京→济南、北京→南京、北京→杭州、北京→上海
- 济南→南京、济南→杭州、济南→上海
- 南京→杭州、南京→上海
- 杭州→上海

5个站就有 `C(5,2) = 10` 种组合，20个站就有 `C(20,2) = 190` 种！

而且这些库存是**相互关联**的：卖了"北京→上海"全程票后，所有中间区间的库存都要减少。

**位图方案的优势：**

用位图只需要存储 **N-1 个 bit**（N为站数），一个 int64 可以表示最多 65 个站的列车。

### 4.2 核心概念

#### 4.2.1 区间段定义

列车有 N 个站，就有 **N-1 个区间段**。每个区间段用一个 bit 表示：

```
站点:    北京    济南    南京    杭州    上海
站序:     0       1       2       3       4
          |-------|-------|-------|-------|
区间段:      0       1       2       3
位位置:    bit0    bit1    bit2    bit3
```

#### 4.2.2 座位状态位图

每个座位用一个整数（uint64）表示状态，**每个 bit 代表一个区间段是否被占用**：

| 状态值 | 二进制 | 含义 |
|-------|--------|------|
| 0 | 0b0000 | 全程空闲，可售任意区间 |
| 1 | 0b0001 | 区间0(北京→济南)已售 |
| 2 | 0b0010 | 区间1(济南→南京)已售 |
| 3 | 0b0011 | 区间0和1已售(北京→南京已售) |
| 5 | 0b0101 | 区间0和2已售(北京→济南 + 南京→杭州) |
| 15 | 0b1111 | 全程已售，不可再售 |

**同一个座位可以在不同区间卖给不同的人：**

```
5车厢10A座位:
- 北京→济南：卖给乘客A   (区间0)
- 济南→南京：卖给乘客B   (区间1)
- 南京→上海：卖给乘客C   (区间2,3)

座位状态变化:
初始:       0b0000 = 0
卖出区间0:   0b0001 = 1
卖出区间1:   0b0011 = 3
卖出区间2,3: 0b1111 = 15 (满)
```

#### 4.2.3 购票区间掩码

乘客要买 `出发站 → 到达站` 的票，需要生成对应的**区间掩码**：

```
北京(0) → 上海(4): 占用区间 0,1,2,3 → 掩码 = 0b1111 = 15
北京(0) → 南京(2): 占用区间 0,1     → 掩码 = 0b0011 = 3
济南(1) → 杭州(3): 占用区间 1,2     → 掩码 = 0b0110 = 6
杭州(3) → 上海(4): 占用区间 3       → 掩码 = 0b1000 = 8
```

### 4.3 掩码计算公式

```
掩码 = ((1 << (到达站序 - 出发站序)) - 1) << 出发站序
```

**推导过程：**

```
1. (到达站序 - 出发站序) = 区间段数量
2. (1 << 区间段数量) - 1 = 生成"区间段数量"个连续的1
3. 左移出发站序位 = 将这些1移动到正确的位置

举例: 济南(1) → 杭州(3)
区间段数量 = 3 - 1 = 2
生成2个1: (1 << 2) - 1 = 4 - 1 = 3 = 0b11
左移1位:  3 << 1 = 6 = 0b110

验证: 济南→杭州 占用区间1和区间2，对应 bit1 和 bit2
0b110 = bit1=1, bit2=1 ✓
```

### 4.4 核心位运算操作

#### 4.4.1 判断座位是否可售

```go
// 可售条件: 购票区间与已占用区间无冲突
可售 = (座位状态 & 购票掩码) == 0
```

**示例：**

```
座位状态 = 0b0101 (北京→济南 和 南京→杭州 已售)

查询: 济南→南京 (掩码 = 0b0010) 是否可售？
判断: 0b0101 & 0b0010 = 0b0000 = 0 ✓ 可售

查询: 北京→南京 (掩码 = 0b0011) 是否可售？
判断: 0b0101 & 0b0011 = 0b0001 ≠ 0 ✗ 不可售(区间0冲突)

查询: 杭州→上海 (掩码 = 0b1000) 是否可售？
判断: 0b0101 & 0b1000 = 0b0000 = 0 ✓ 可售
```

#### 4.4.2 购票更新状态

```go
// 购票: 将购票区间标记为已占用
新状态 = 原状态 | 购票掩码
```

**示例：**

```
原状态 = 0b0101
购买: 济南→南京，掩码 = 0b0010
新状态 = 0b0101 | 0b0010 = 0b0111
```

#### 4.4.3 退票更新状态

```go
// 退票: 清除对应区间的占用标记
新状态 = 原状态 &^ 退票掩码   // Go语言的位清除操作
// 等价于
新状态 = 原状态 & (^退票掩码)
```

**示例：**

```
原状态 = 0b0111
退票: 济南→南京，掩码 = 0b0010
^掩码 = ^0b0010 = 0b1101
新状态 = 0b0111 & 0b1101 = 0b0101
```

### 4.5 多车次查询处理

#### 4.5.1 场景说明

用户查询：`北京 → 上海`，日期 `2024-01-15`

数据库中有 3 趟车经过这两站：

| 车次 | 路线 | 北京站序 | 上海站序 | 总站数 |
|-----|------|---------|---------|-------|
| G1 | 北京→济南→南京→上海 | 0 | 3 | 4 |
| G2 | 北京→天津→南京→杭州→上海 | 0 | 4 | 5 |
| G3 | 石家庄→北京→上海 | 1 | 2 | 3 |

#### 4.5.2 掩码计算

**关键点：每趟车的位图是独立的，掩码基于该车次内的站序计算。**

```
G1: 北京(0) → 上海(3)
    掩码 = ((1 << 3) - 1) << 0 = 7 = 0b0111
    该车有3个区间段 (0,1,2)，购买全程需占用全部

G2: 北京(0) → 上海(4)
    掩码 = ((1 << 4) - 1) << 0 = 15 = 0b1111
    该车有4个区间段 (0,1,2,3)

G3: 北京(1) → 上海(2)
    掩码 = ((1 << 1) - 1) << 1 = 1 << 1 = 2 = 0b10
    该车有2个区间段，北京→上海只占区间1
```

#### 4.5.3 查询流程图

```
┌─────────────────────────────────────────────────────────────┐
│                    余票查询流程                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  输入: 出发城市=北京, 到达城市=上海, 日期=2024-01-15          │
│                                                             │
│  Step 1: 查找经过两站的车次                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ SELECT DISTINCT t.*, ts1.stop_order, ts2.stop_order │   │
│  │ FROM trains t                                        │   │
│  │ JOIN train_stops ts1 ON t.id = ts1.train_id         │   │
│  │ JOIN train_stops ts2 ON t.id = ts2.train_id         │   │
│  │ JOIN stations s1 ON ts1.station_id = s1.id          │   │
│  │ JOIN stations s2 ON ts2.station_id = s2.id          │   │
│  │ WHERE s1.name = '北京' AND s2.name = '上海'          │   │
│  │   AND ts1.stop_order < ts2.stop_order  ← 确保方向正确 │   │
│  └─────────────────────────────────────────────────────┘   │
│                          │                                  │
│        结果: G1(站序0→3), G2(站序0→4), G3(站序1→2)          │
│                          │                                  │
│                          ▼                                  │
│  Step 2: 对每个车次计算掩码并统计余票                         │
│                                                             │
│  ┌─────────── G1 ───────────┐                              │
│  │ 掩码 = 0b0111            │                              │
│  │ SELECT COUNT(*) FROM ... │                              │
│  │ WHERE (status & 7) = 0   │                              │
│  │ 结果: 二等座 500张        │                              │
│  └──────────────────────────┘                              │
│                                                             │
│  ┌─────────── G2 ───────────┐                              │
│  │ 掩码 = 0b1111            │                              │
│  │ SELECT COUNT(*) FROM ... │                              │
│  │ WHERE (status & 15) = 0  │                              │
│  │ 结果: 二等座 320张        │                              │
│  └──────────────────────────┘                              │
│                                                             │
│  ┌─────────── G3 ───────────┐                              │
│  │ 掩码 = 0b10              │                              │
│  │ SELECT COUNT(*) FROM ... │                              │
│  │ WHERE (status & 2) = 0   │                              │
│  │ 结果: 二等座 450张        │                              │
│  └──────────────────────────┘                              │
│                                                             │
│  Step 3: 返回结果                                           │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ G1  08:00-12:30  4h30m  二等座¥553 (500张)           │  │
│  │ G2  09:00-14:00  5h00m  二等座¥553 (320张)           │  │
│  │ G3  10:00-15:30  5h30m  二等座¥553 (450张)           │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 4.6 位图算法 Go 代码实现

```go
package ticket

import (
    "errors"
    "fmt"
)

// ===================== 基础位运算函数 =====================

// CalcSegmentMask 计算购票区间掩码
// departOrder: 出发站在该车次中的站序（从0开始）
// arriveOrder: 到达站在该车次中的站序
// 返回: 区间掩码
func CalcSegmentMask(departOrder, arriveOrder int) uint64 {
    if departOrder >= arriveOrder {
        return 0 // 无效区间
    }
    // 公式: ((1 << 区间数) - 1) << 起始位置
    segmentCount := arriveOrder - departOrder
    return uint64((1 << segmentCount) - 1) << departOrder
}

// IsSeatAvailable 判断座位在指定区间是否可售
// seatStatus: 座位当前状态位图
// mask: 购票区间掩码
// 返回: true=可售, false=不可售
func IsSeatAvailable(seatStatus, mask uint64) bool {
    // 位与操作，如果结果为0说明无冲突
    return (seatStatus & mask) == 0
}

// BookSeat 购票后更新座位状态
// currentStatus: 当前状态
// mask: 购票区间掩码
// 返回: 新状态
func BookSeat(currentStatus, mask uint64) uint64 {
    // 位或操作，将购票区间标记为已占用
    return currentStatus | mask
}

// RefundSeat 退票后更新座位状态
// currentStatus: 当前状态
// mask: 退票区间掩码
// 返回: 新状态
func RefundSeat(currentStatus, mask uint64) uint64 {
    // 位清除操作，清除对应区间的占用标记
    return currentStatus &^ mask
}

// ===================== 业务层函数 =====================

// SeatStatus 座位状态信息
type SeatStatus struct {
    SeatID   int64  // 座位ID
    SeatNo   string // 座位号，如 "05A"
    SeatType string // 席别：商务座/一等座/二等座
    Status   uint64 // 位图状态
}

// TrainStopInfo 车次经停信息
type TrainStopInfo struct {
    TrainID       int64
    TrainNo       string // 车次号
    DepartStation string // 出发站名
    ArriveStation string // 到达站名
    DepartOrder   int    // 出发站序
    ArriveOrder   int    // 到达站序
    DepartTime    string // 出发时间
    ArriveTime    string // 到达时间
}

// TicketCountResult 余票统计结果
type TicketCountResult struct {
    TrainNo       string
    DepartStation string
    ArriveStation string
    DepartTime    string
    ArriveTime    string
    SeatCounts    map[string]int // 席别 -> 余票数
}

// CountAvailableSeats 统计某车次某区间各席别的可售座位数
// seatStatuses: 该车次所有座位的状态列表
// departOrder: 出发站序
// arriveOrder: 到达站序
// 返回: 各席别的余票数 map[席别]数量
func CountAvailableSeats(seatStatuses []SeatStatus, departOrder, arriveOrder int) map[string]int {
    mask := CalcSegmentMask(departOrder, arriveOrder)
    result := make(map[string]int)

    for _, seat := range seatStatuses {
        if IsSeatAvailable(seat.Status, mask) {
            result[seat.SeatType]++
        }
    }
    return result
}

// FindAvailableSeats 查找可售座位列表
// seatStatuses: 座位状态列表
// departOrder: 出发站序
// arriveOrder: 到达站序
// seatType: 席别筛选（空字符串表示不筛选）
// 返回: 可售座位列表
func FindAvailableSeats(seatStatuses []SeatStatus, departOrder, arriveOrder int, seatType string) []SeatStatus {
    mask := CalcSegmentMask(departOrder, arriveOrder)
    var result []SeatStatus

    for _, seat := range seatStatuses {
        // 检查席别
        if seatType != "" && seat.SeatType != seatType {
            continue
        }
        // 检查是否可售
        if IsSeatAvailable(seat.Status, mask) {
            result = append(result, seat)
        }
    }
    return result
}

// ===================== 示例演示 =====================

// DemoMaskCalculation 演示掩码计算
func DemoMaskCalculation() {
    fmt.Println("=== 掩码计算演示 ===")
    fmt.Println("车次: 北京(0) → 济南(1) → 南京(2) → 杭州(3) → 上海(4)")
    fmt.Println()

    cases := []struct {
        depart string
        arrive string
        dOrder int
        aOrder int
    }{
        {"北京", "上海", 0, 4},
        {"北京", "南京", 0, 2},
        {"济南", "杭州", 1, 3},
        {"杭州", "上海", 3, 4},
        {"济南", "南京", 1, 2},
    }

    for _, c := range cases {
        mask := CalcSegmentMask(c.dOrder, c.aOrder)
        fmt.Printf("%s(%d) → %s(%d): 掩码 = %d = %04b\n",
            c.depart, c.dOrder, c.arrive, c.aOrder, mask, mask)
    }
}

// DemoSeatAvailability 演示座位可售性判断
func DemoSeatAvailability() {
    fmt.Println("\n=== 座位可售性判断演示 ===")

    // 座位状态: 北京→济南(区间0) 和 南京→杭州(区间2) 已售
    seatStatus := uint64(0b0101) // = 5
    fmt.Printf("座位当前状态: %d = %04b\n", seatStatus, seatStatus)
    fmt.Println("已售区间: 北京→济南(bit0), 南京→杭州(bit2)")
    fmt.Println()

    queries := []struct {
        depart string
        arrive string
        dOrder int
        aOrder int
    }{
        {"济南", "南京", 1, 2},
        {"北京", "南京", 0, 2},
        {"杭州", "上海", 3, 4},
        {"北京", "上海", 0, 4},
    }

    for _, q := range queries {
        mask := CalcSegmentMask(q.dOrder, q.aOrder)
        available := IsSeatAvailable(seatStatus, mask)
        result := "✗ 不可售"
        if available {
            result = "✓ 可售"
        }
        fmt.Printf("%s→%s (掩码=%04b): %d & %d = %d → %s\n",
            q.depart, q.arrive, mask,
            seatStatus, mask, seatStatus&mask, result)
    }
}

// DemoBookAndRefund 演示购票和退票
func DemoBookAndRefund() {
    fmt.Println("\n=== 购票和退票演示 ===")

    status := uint64(0)
    fmt.Printf("初始状态: %d = %04b\n", status, status)

    // 购票: 北京→济南
    mask1 := CalcSegmentMask(0, 1)
    status = BookSeat(status, mask1)
    fmt.Printf("购票 北京→济南 (掩码=%04b): 新状态 = %d = %04b\n", mask1, status, status)

    // 购票: 南京→上海
    mask2 := CalcSegmentMask(2, 4)
    status = BookSeat(status, mask2)
    fmt.Printf("购票 南京→上海 (掩码=%04b): 新状态 = %d = %04b\n", mask2, status, status)

    // 退票: 北京→济南
    status = RefundSeat(status, mask1)
    fmt.Printf("退票 北京→济南 (掩码=%04b): 新状态 = %d = %04b\n", mask1, status, status)
}
```

### 4.7 座位位图生命周期管理

#### 4.7.1 每个座位独立维护位图

每个座位都有自己专属的位图状态：

```
D123 次列车，5车厢：

座位 05A → 位图状态 status = 0b0101
座位 05B → 位图状态 status = 0b0000
座位 05C → 位图状态 status = 0b0011
座位 05D → 位图状态 status = 0b0110
...
```

#### 4.7.2 按日期分开存储（而非"清0"）

**常见误解**：列车运行完后把所有座位状态清0。

**实际设计**：按日期隔离存储，每个日期的数据相互独立。

```sql
-- daily_seat_status 表结构
CREATE TABLE daily_seat_status (
    seat_id BIGINT,
    departure_date DATE,      -- 关键：按发车日期区分
    status BIGINT DEFAULT 0,
    PRIMARY KEY (seat_id, departure_date)
);
```

**数据示例：**

```
┌─────────────────────────────────────────────────────────┐
│                    D123 - 座位 05A                       │
├─────────────────────────────────────────────────────────┤
│  2024-01-15  │  status = 0b1111  (已售完)               │
│  2024-01-16  │  status = 0b0101  (部分已售)             │
│  2024-01-17  │  status = 0b0000  (全空，可预售)         │
│  2024-01-18  │  status = 0b0000  (全空，可预售)         │
│  ...         │                                          │
└─────────────────────────────────────────────────────────┘
```

#### 4.7.3 为什么不用"清0"方案？

| 方案 | 说明 | 问题 |
|-----|------|------|
| 清0 | 每天凌晨把所有座位状态重置为0 | 无法提前售票（12306可预售15天） |
| 按日期分表 | 每个日期独立存储 | ✓ 支持预售，数据隔离清晰 |

#### 4.7.4 数据生命周期

```
                    时间线
    ◄────────────────────────────────────────►

    15天前        今天         15天后
      │            │             │
      ▼            ▼             ▼
┌──────────┐  ┌──────────┐  ┌──────────┐
│ 历史数据  │  │ 今日班次  │  │ 预售数据  │
│ 可归档    │  │ 正在运行  │  │ status=0 │
│ 或删除    │  │          │  │ 等待售票  │
└──────────┘  └──────────┘  └──────────┘
```

#### 4.7.5 定时任务处理

```go
// 每日凌晨执行

// 1. 为新的预售日期初始化座位状态（全0）
func InitDailySeats(db *sql.DB, date time.Time) error {
    query := `
        INSERT INTO daily_seat_status (seat_id, departure_date, status)
        SELECT id, ?, 0 FROM seats
    `
    _, err := db.Exec(query, date.Format("2006-01-02"))
    return err
}

// 2. 归档/删除过期数据
func ArchiveOldData(db *sql.DB, beforeDate time.Time) error {
    // 方案1: 直接删除
    query := `DELETE FROM daily_seat_status WHERE departure_date < ?`
    _, err := db.Exec(query, beforeDate.Format("2006-01-02"))
    return err

    // 方案2: 移动到归档表（用于对账、统计分析）
    // INSERT INTO daily_seat_status_archive SELECT * FROM daily_seat_status WHERE ...
    // DELETE FROM daily_seat_status WHERE ...
}
```

#### 4.7.6 座位位图的完整生命周期

```
┌─────────────────────────────────────────────────────────────┐
│                     D123 座位 05A 的一生                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  T-15天 (预售开始)                                          │
│  ┌─────────────────────────────┐                           │
│  │ INSERT (seat_05A, 1月15日, 0) │  ← 初始化，全空          │
│  └─────────────────────────────┘                           │
│                                                             │
│  T-10天 (有人买票：北京→济南)                                │
│  ┌─────────────────────────────┐                           │
│  │ status = 0 | 0b01 = 0b01    │  ← 区间0被占用            │
│  └─────────────────────────────┘                           │
│                                                             │
│  T-5天 (有人买票：济南→南京)                                 │
│  ┌─────────────────────────────┐                           │
│  │ status = 0b01 | 0b10 = 0b11 │  ← 区间0,1都被占用        │
│  └─────────────────────────────┘                           │
│                                                             │
│  T-1天 (有人退票：北京→济南)                                 │
│  ┌─────────────────────────────┐                           │
│  │ status = 0b11 &^ 0b01 = 0b10│  ← 区间0释放              │
│  └─────────────────────────────┘                           │
│                                                             │
│  T日 08:00 (列车发车)                                       │
│  ┌─────────────────────────────┐                           │
│  │ 停止售票，数据保留用于对账    │                           │
│  └─────────────────────────────┘                           │
│                                                             │
│  T+30天 (数据归档)                                          │
│  ┌─────────────────────────────┐                           │
│  │ DELETE 或移动到历史表        │                           │
│  └─────────────────────────────┘                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 4.7.7 常见问题总结

| 问题 | 答案 |
|-----|------|
| 每个座位有专属位图吗？ | ✓ 是的，每个座位独立维护 |
| 运行完清0吗？ | ✗ 不是清0，是按日期分开存储 |
| 同一座位不同日期的数据？ | 完全独立，互不影响 |
| 历史数据怎么处理？ | 定期归档或删除 |
| 为什么不清0？ | 需要支持预售（提前15天售票） |

---

## 五、选座算法详解

### 5.1 座位布局

#### 5.1.1 高铁座位布局图

```
二等座 (3+2布局，每排5座):
    窗 |     | 过道 |     | 窗
    A     B     C   |   D     F
    ─────────────────────────────
01  A     B     C   |   D     F
02  A     B     C   |   D     F
03  A     B     C   |   D     F
...
18  A     B     C   |   D     F

一等座 (2+2布局，每排4座):
    窗 | 过道 | 过道 | 窗
    A     C   |   D     F
    ─────────────────────
01  A     C   |   D     F
02  A     C   |   D     F
...

商务座 (2+1布局，每排3座):
    窗 | 过道 |     | 窗
    A     C   |       F
    ─────────────────────
01  A     C   |       F
02  A     C   |       F
...
```

#### 5.1.2 座位位置分类

| 席别 | 靠窗 | 靠过道 | 中间 |
|-----|------|-------|------|
| 二等座 | A, F | C, D | B |
| 一等座 | A, F | C, D | - |
| 商务座 | A, F | C | - |

### 5.2 座位数据结构

```go
// Seat 座位信息
type Seat struct {
    ID         int64
    CarriageID int64
    SeatNo     string // 如 "05A", "12F"
    Row        int    // 排号: 5, 12
    Column     string // 列号: A, B, C, D, F
    SeatType   string // 商务座/一等座/二等座
    Position   string // window(靠窗) / aisle(靠过道) / middle(中间)
}

// SeatLayout 座位布局配置
type SeatLayout struct {
    SeatType   string   // 席别
    Columns    []string // 列序: ["A","B","C","D","F"]
    RowCount   int      // 每车厢排数
    WindowCols []string // 靠窗列: ["A","F"]
    AisleCols  []string // 靠过道列: ["C","D"]
    MiddleCols []string // 中间列: ["B"]
    GroupLeft  []string // 左侧组: ["A","B","C"]
    GroupRight []string // 右侧组: ["D","F"]
}

// 预定义布局
var (
    SecondClassLayout = SeatLayout{
        SeatType:   "二等座",
        Columns:    []string{"A", "B", "C", "D", "F"},
        RowCount:   18,
        WindowCols: []string{"A", "F"},
        AisleCols:  []string{"C", "D"},
        MiddleCols: []string{"B"},
        GroupLeft:  []string{"A", "B", "C"},
        GroupRight: []string{"D", "F"},
    }

    FirstClassLayout = SeatLayout{
        SeatType:   "一等座",
        Columns:    []string{"A", "C", "D", "F"},
        RowCount:   13,
        WindowCols: []string{"A", "F"},
        AisleCols:  []string{"C", "D"},
        MiddleCols: []string{},
        GroupLeft:  []string{"A", "C"},
        GroupRight: []string{"D", "F"},
    }

    BusinessClassLayout = SeatLayout{
        SeatType:   "商务座",
        Columns:    []string{"A", "C", "F"},
        RowCount:   8,
        WindowCols: []string{"A", "F"},
        AisleCols:  []string{"C"},
        MiddleCols: []string{},
        GroupLeft:  []string{"A", "C"},
        GroupRight: []string{"F"},
    }
)
```

### 5.3 选座偏好

```go
// SeatPreference 选座偏好
type SeatPreference struct {
    PreferWindow bool     // 优先靠窗
    PreferAisle  bool     // 优先靠过道
    SpecificSeat string   // 指定座位号，如 "05A"
    NeedAdjacent bool     // 需要连座（多人时）
}
```

### 5.4 单人选座算法

```go
// SelectSingleSeat 单人选座
// availableSeats: 可售座位列表
// pref: 选座偏好
// 返回: 选中的座位，nil表示无合适座位
func SelectSingleSeat(availableSeats []Seat, pref SeatPreference) *Seat {
    if len(availableSeats) == 0 {
        return nil
    }

    // 1. 指定座位优先
    if pref.SpecificSeat != "" {
        for i := range availableSeats {
            if availableSeats[i].SeatNo == pref.SpecificSeat {
                return &availableSeats[i]
            }
        }
        return nil // 指定座位不可用
    }

    // 2. 按偏好筛选
    var preferred []Seat
    for _, seat := range availableSeats {
        if pref.PreferWindow && seat.Position == "window" {
            preferred = append(preferred, seat)
        } else if pref.PreferAisle && seat.Position == "aisle" {
            preferred = append(preferred, seat)
        }
    }

    // 3. 有符合偏好的，选排号最小的
    if len(preferred) > 0 {
        sortByRow(preferred)
        return &preferred[0]
    }

    // 4. 无符合偏好的，返回任意可用座位（排号最小）
    sortByRow(availableSeats)
    return &availableSeats[0]
}
```

### 5.5 多人连座算法（核心）

#### 5.5.1 算法流程

```
                     ┌─────────────────┐
                     │   用户提交订单   │
                     │ 3人，偏好靠窗    │
                     └────────┬────────┘
                              │
                              ▼
                     ┌─────────────────┐
                     │  查询可用座位    │
                     │ (位图 & 掩码)=0  │
                     └────────┬────────┘
                              │
                              ▼
              ┌───────────────────────────────┐
              │      可用座位数 >= 3 ?         │
              └───────────────┬───────────────┘
                    No        │        Yes
                    │         │         │
                    ▼         │         ▼
              ┌──────────┐    │  ┌─────────────────┐
              │ 进入候补  │    │  │ 策略1: 同排连座  │
              │ 或提示不足│    │  │  (最优方案)      │
              └──────────┘    │  └────────┬────────┘
                              │           │
                              │     Found │ Not Found
                              │       │   │
                              │       │   ▼
                              │       │  ┌─────────────────┐
                              │       │  │ 策略2: 相邻排   │
                              │       │  │  (次优方案)      │
                              │       │  └────────┬────────┘
                              │       │           │
                              │       │     Found │ Not Found
                              │       │       │   │
                              │       │       │   ▼
                              │       │       │  ┌─────────────────┐
                              │       │       │  │ 策略3: 分散分配 │
                              │       │       │  │  (保底方案)      │
                              │       │       │  └────────┬────────┘
                              │       │       │           │
                              │       ▼       ▼           ▼
                              │  ┌────────────────────────────┐
                              │  │      应用偏好过滤           │
                              │  │  (在已选座位中优先靠窗)     │
                              │  └─────────────┬──────────────┘
                              │                │
                              ▼                ▼
                     ┌─────────────────────────────────┐
                     │           返回选座结果           │
                     │  [{05A, 05B, 05C}, 同排连座]    │
                     └─────────────────────────────────┘
```

#### 5.5.2 代码实现

```go
// ConsecutiveResult 连座选择结果
type ConsecutiveResult struct {
    Seats    []Seat
    SameRow  bool // 是否同排
    Adjacent bool // 是否相邻
}

// SelectConsecutiveSeats 多人连座选择
// availableSeats: 可售座位列表
// count: 需要的座位数
// layout: 座位布局配置
// 返回: 连座结果
func SelectConsecutiveSeats(
    availableSeats []Seat,
    count int,
    layout SeatLayout,
) *ConsecutiveResult {
    if count <= 0 || len(availableSeats) < count {
        return nil
    }

    // 按排号分组
    rowSeats := groupByRow(availableSeats)

    // 策略1: 同排连座（最优）
    result := findSameRowConsecutive(rowSeats, count, layout)
    if result != nil {
        return result
    }

    // 策略2: 相邻排（次优）
    result = findAdjacentRowSeats(rowSeats, count, layout)
    if result != nil {
        return result
    }

    // 策略3: 分散分配（保底）
    return findAnyAvailable(availableSeats, count)
}

// groupByRow 按排号分组
func groupByRow(seats []Seat) map[int][]Seat {
    result := make(map[int][]Seat)
    for _, seat := range seats {
        result[seat.Row] = append(result[seat.Row], seat)
    }
    return result
}

// findSameRowConsecutive 同排连座查找
func findSameRowConsecutive(
    rowSeats map[int][]Seat,
    count int,
    layout SeatLayout,
) *ConsecutiveResult {

    // 列顺序映射 (用于判断相邻)
    colOrder := map[string]int{"A": 0, "B": 1, "C": 2, "D": 3, "F": 4}

    // 遍历每一排（从小排号开始）
    for row := 1; row <= layout.RowCount; row++ {
        seats, ok := rowSeats[row]
        if !ok || len(seats) < count {
            continue
        }

        // 尝试在左侧组 (A,B,C) 找连座
        leftSeats := filterByColumns(seats, layout.GroupLeft)
        if consecutive := findConsecutiveInGroup(leftSeats, count, colOrder); consecutive != nil {
            return &ConsecutiveResult{
                Seats:    consecutive,
                SameRow:  true,
                Adjacent: true,
            }
        }

        // 尝试在右侧组 (D,F) 找连座
        rightSeats := filterByColumns(seats, layout.GroupRight)
        if consecutive := findConsecutiveInGroup(rightSeats, count, colOrder); consecutive != nil {
            return &ConsecutiveResult{
                Seats:    consecutive,
                SameRow:  true,
                Adjacent: true,
            }
        }

        // 特殊情况: 2人可选 C+D（过道两侧，视为相邻）
        if count == 2 {
            c := findByColumn(seats, "C")
            d := findByColumn(seats, "D")
            if c != nil && d != nil {
                return &ConsecutiveResult{
                    Seats:    []Seat{*c, *d},
                    SameRow:  true,
                    Adjacent: true,
                }
            }
        }
    }

    return nil
}

// filterByColumns 按列筛选座位
func filterByColumns(seats []Seat, cols []string) []Seat {
    colSet := make(map[string]bool)
    for _, c := range cols {
        colSet[c] = true
    }

    var result []Seat
    for _, seat := range seats {
        if colSet[seat.Column] {
            result = append(result, seat)
        }
    }
    return result
}

// findConsecutiveInGroup 在一组座位中找连续的
func findConsecutiveInGroup(seats []Seat, count int, colOrder map[string]int) []Seat {
    if len(seats) < count {
        return nil
    }

    // 按列排序
    sortByColumn(seats, colOrder)

    // 滑动窗口找连续座位
    for i := 0; i <= len(seats)-count; i++ {
        consecutive := true
        for j := 1; j < count; j++ {
            prevOrder := colOrder[seats[i+j-1].Column]
            currOrder := colOrder[seats[i+j].Column]
            // 注意: D(3) 到 F(4) 之间没有E，但仍视为连续
            if currOrder-prevOrder != 1 {
                consecutive = false
                break
            }
        }
        if consecutive {
            result := make([]Seat, count)
            copy(result, seats[i:i+count])
            return result
        }
    }

    return nil
}

// findByColumn 按列号查找座位
func findByColumn(seats []Seat, col string) *Seat {
    for i := range seats {
        if seats[i].Column == col {
            return &seats[i]
        }
    }
    return nil
}

// findAdjacentRowSeats 相邻排座位查找
func findAdjacentRowSeats(
    rowSeats map[int][]Seat,
    count int,
    layout SeatLayout,
) *ConsecutiveResult {
    if count <= 1 {
        return nil
    }

    colOrder := map[string]int{"A": 0, "B": 1, "C": 2, "D": 3, "F": 4}

    // 尝试拆分: (count-1)+1, (count-2)+2, ...
    // 例如3人: 先尝试2+1，保证至少2人同排
    for split := count - 1; split >= (count+1)/2; split-- {
        remain := count - split

        // 找相邻的两排
        for row := 1; row < layout.RowCount; row++ {
            seats1, ok1 := rowSeats[row]
            seats2, ok2 := rowSeats[row+1]
            if !ok1 || !ok2 {
                continue
            }

            // 在row排找split个连座
            var group1 []Seat
            leftSeats := filterByColumns(seats1, layout.GroupLeft)
            group1 = findConsecutiveInGroup(leftSeats, split, colOrder)

            usedLeft := group1 != nil
            if group1 == nil {
                rightSeats := filterByColumns(seats1, layout.GroupRight)
                group1 = findConsecutiveInGroup(rightSeats, split, colOrder)
            }

            if group1 == nil {
                continue
            }

            // 在row+1排找remain个，优先同侧
            var targetCols []string
            if usedLeft {
                targetCols = layout.GroupLeft
            } else {
                targetCols = layout.GroupRight
            }

            targetSeats := filterByColumns(seats2, targetCols)
            group2 := findConsecutiveInGroup(targetSeats, remain, colOrder)

            // 同侧没有，尝试另一侧
            if group2 == nil {
                if usedLeft {
                    targetCols = layout.GroupRight
                } else {
                    targetCols = layout.GroupLeft
                }
                targetSeats = filterByColumns(seats2, targetCols)
                group2 = findConsecutiveInGroup(targetSeats, remain, colOrder)
            }

            if group2 != nil {
                allSeats := append(group1, group2...)
                return &ConsecutiveResult{
                    Seats:    allSeats,
                    SameRow:  false,
                    Adjacent: true,
                }
            }
        }
    }

    return nil
}

// findAnyAvailable 分散分配（保底方案）
func findAnyAvailable(seats []Seat, count int) *ConsecutiveResult {
    if len(seats) < count {
        return nil
    }

    // 按排号排序，取前count个
    sortByRow(seats)
    result := make([]Seat, count)
    copy(result, seats[:count])

    return &ConsecutiveResult{
        Seats:    result,
        SameRow:  false,
        Adjacent: false,
    }
}

// sortByRow 按排号排序
func sortByRow(seats []Seat) {
    sort.Slice(seats, func(i, j int) bool {
        if seats[i].Row != seats[j].Row {
            return seats[i].Row < seats[j].Row
        }
        return seats[i].Column < seats[j].Column
    })
}

// sortByColumn 按列排序
func sortByColumn(seats []Seat, colOrder map[string]int) {
    sort.Slice(seats, func(i, j int) bool {
        return colOrder[seats[i].Column] < colOrder[seats[j].Column]
    })
}
```

### 5.6 选座优化策略

#### 5.6.1 座位价值评分

不同座位有不同的"价值"，优化策略可以提高整体售出率：

```go
// CalcSeatScore 计算座位价值评分（分数越低越优先分配）
// seat: 座位信息
// isShortTrip: 是否短途行程
func CalcSeatScore(seat Seat, isShortTrip bool) int {
    score := 0

    if isShortTrip {
        // 短途票优先分配中间座位，保留好座位给长途
        switch seat.Position {
        case "middle":
            score += 0 // 最优先分配
        case "aisle":
            score += 10
        case "window":
            score += 20 // 最后分配
        }
    } else {
        // 长途票按用户偏好正常分配
        switch seat.Position {
        case "window":
            score += 0 // 靠窗优先
        case "aisle":
            score += 5
        case "middle":
            score += 10
        }
    }

    // 排号越小越优先（靠近车门）
    score += seat.Row

    return score
}
```

#### 5.6.2 碎片座位优先

已有部分区间被占用的座位，应优先分配给短途乘客：

```go
// CalcFragmentation 计算座位碎片度（已售区间数）
func CalcFragmentation(seatStatus uint64) int {
    count := 0
    for seatStatus > 0 {
        count += int(seatStatus & 1)
        seatStatus >>= 1
    }
    return count
}

// SeatWithStatus 带状态的座位
type SeatWithStatus struct {
    Seat
    Status uint64
}

// SelectForShortTrip 短途票优先分配碎片座位
func SelectForShortTrip(seats []SeatWithStatus, mask uint64) *Seat {
    // 按碎片度降序排列（碎片多的优先）
    sort.Slice(seats, func(i, j int) bool {
        return CalcFragmentation(seats[i].Status) > CalcFragmentation(seats[j].Status)
    })

    for _, s := range seats {
        if (s.Status & mask) == 0 {
            return &s.Seat
        }
    }
    return nil
}
```

---

## 六、高并发设计

### 6.1 库存扣减方案

#### 6.1.1 Redis 预扣减 + 数据库最终一致

```
┌─────────────────────────────────────────────────────────────┐
│                      锁票流程                                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Redis 原子操作: DECR ticket:count:{车次}:{区间}:{席别}   │
│                          │                                  │
│            ┌─────────────┴─────────────┐                   │
│            │                           │                    │
│      结果 >= 0                   结果 < 0                   │
│            │                           │                    │
│            ▼                           ▼                    │
│     锁票成功，进入支付           INCR 回滚，返回库存不足       │
│            │                                                │
│            ▼                                                │
│  2. 创建待支付订单（数据库）                                  │
│                                                             │
│  3. 用户支付                                                 │
│            │                                                │
│     ┌──────┴──────┐                                        │
│     │             │                                         │
│  支付成功      支付超时/取消                                  │
│     │             │                                         │
│     ▼             ▼                                         │
│  消息队列异步    Redis: INCR 恢复库存                         │
│  更新数据库     释放座位锁定                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 6.1.2 Redis Lua 脚本保证原子性

```lua
-- 扣减库存的 Lua 脚本
-- KEYS[1]: 库存key
-- ARGV[1]: 扣减数量
local count = redis.call('GET', KEYS[1])
if count == false then
    return -1  -- key不存在
end
if tonumber(count) >= tonumber(ARGV[1]) then
    redis.call('DECRBY', KEYS[1], ARGV[1])
    return 1  -- 成功
else
    return 0  -- 库存不足
end
```

#### 6.1.3 Go 代码实现

```go
// 扣减库存的 Lua 脚本
const decrStockScript = `
local count = redis.call('GET', KEYS[1])
if count == false then
    return -1
end
if tonumber(count) >= tonumber(ARGV[1]) then
    redis.call('DECRBY', KEYS[1], ARGV[1])
    return 1
else
    return 0
end
`

// DecrStock 扣减库存
func DecrStock(ctx context.Context, rdb *redis.Client,
    trainID int64, date string, segment string, seatType string, count int) (bool, error) {

    key := fmt.Sprintf("ticket:count:%d:%s:%s:%s", trainID, date, segment, seatType)

    result, err := rdb.Eval(ctx, decrStockScript, []string{key}, count).Int()
    if err != nil {
        return false, err
    }

    switch result {
    case 1:
        return true, nil  // 成功
    case 0:
        return false, nil // 库存不足
    default:
        return false, errors.New("库存数据不存在")
    }
}

// IncrStock 恢复库存（退票或超时）
func IncrStock(ctx context.Context, rdb *redis.Client,
    trainID int64, date string, segment string, seatType string, count int) error {

    key := fmt.Sprintf("ticket:count:%d:%s:%s:%s", trainID, date, segment, seatType)
    return rdb.IncrBy(ctx, key, int64(count)).Err()
}
```

### 6.2 分布式锁

```go
// 选座时的分布式锁
func SelectSeatsWithLock(
    ctx context.Context,
    rdb *redis.Client,
    trainID int64,
    date string,
    count int,
    pref SeatPreference,
) ([]Seat, error) {

    // 1. 获取车次级别的分布式锁
    lockKey := fmt.Sprintf("lock:select:%d:%s", trainID, date)

    // 尝试获取锁，最多等待3秒，锁持有5秒
    lock := rdb.SetNX(ctx, lockKey, "1", 5*time.Second)
    if !lock.Val() {
        return nil, errors.New("系统繁忙，请重试")
    }
    defer rdb.Del(ctx, lockKey)

    // 2. 查询最新可用座位（从缓存或数据库）
    available, err := QueryAvailableSeats(ctx, trainID, date)
    if err != nil {
        return nil, err
    }

    // 3. 执行选座算法
    result := SelectConsecutiveSeats(available, count, SecondClassLayout)
    if result == nil {
        return nil, errors.New("座位不足")
    }

    // 4. 锁定座位（设置15分钟超时）
    err = LockSeats(ctx, trainID, date, result.Seats, 15*time.Minute)
    if err != nil {
        return nil, err
    }

    return result.Seats, nil
}
```

### 6.3 候补购票

```go
// 加入候补队列
func JoinWaitlist(ctx context.Context, rdb *redis.Client,
    trainID int64, date, segment, seatType string, userID int64) error {

    key := fmt.Sprintf("waitlist:%d:%s:%s:%s", trainID, date, segment, seatType)
    score := float64(time.Now().UnixNano())

    // 有序集合，按时间戳排序
    return rdb.ZAdd(ctx, key, &redis.Z{
        Score:  score,
        Member: userID,
    }).Err()
}

// 候补兑现（定时任务执行）
func ProcessWaitlist(ctx context.Context, rdb *redis.Client, db *sql.DB,
    trainID int64, date, segment, seatType string, availableCount int) error {

    key := fmt.Sprintf("waitlist:%d:%s:%s:%s", trainID, date, segment, seatType)

    // 获取排队最靠前的用户
    users, err := rdb.ZRange(ctx, key, 0, int64(availableCount-1)).Result()
    if err != nil {
        return err
    }

    for _, userIDStr := range users {
        userID, _ := strconv.ParseInt(userIDStr, 10, 64)

        // 尝试为用户分配座位
        err := allocateSeatForWaitlist(ctx, rdb, db, trainID, date, segment, seatType, userID)
        if err != nil {
            continue
        }

        // 分配成功，从队列移除
        rdb.ZRem(ctx, key, userIDStr)

        // 通知用户（发送短信、推送等）
        notifyUser(userID, "候补成功，请在15分钟内完成支付")
    }

    return nil
}
```

---

## 七、完整代码实现

### 7.1 项目结构

```
ticket/
├── cmd/
│   └── server/
│       └── main.go           # 程序入口
├── internal/
│   ├── model/                # 数据模型
│   │   ├── user.go
│   │   ├── train.go
│   │   ├── seat.go
│   │   └── order.go
│   ├── repository/           # 数据访问层
│   │   ├── user_repo.go
│   │   ├── train_repo.go
│   │   └── order_repo.go
│   ├── service/              # 业务逻辑层
│   │   ├── ticket_service.go # 核心购票逻辑
│   │   ├── seat_service.go   # 选座逻辑
│   │   └── order_service.go  # 订单逻辑
│   ├── handler/              # HTTP处理器
│   │   └── ticket_handler.go
│   └── algorithm/            # 算法实现
│       ├── bitmap.go         # 位图算法
│       └── seat_select.go    # 选座算法
├── pkg/
│   ├── redis/                # Redis封装
│   └── lock/                 # 分布式锁
├── config/
│   └── config.go
└── go.mod
```

### 7.2 位图算法完整代码 (algorithm/bitmap.go)

```go
package algorithm

import (
    "fmt"
)

// SegmentMask 区间掩码计算器
type SegmentMask struct {
    TotalStops int // 总站数
}

// NewSegmentMask 创建掩码计算器
func NewSegmentMask(totalStops int) *SegmentMask {
    return &SegmentMask{TotalStops: totalStops}
}

// Calc 计算区间掩码
func (sm *SegmentMask) Calc(departOrder, arriveOrder int) (uint64, error) {
    if departOrder < 0 || arriveOrder > sm.TotalStops-1 {
        return 0, fmt.Errorf("站序超出范围: depart=%d, arrive=%d, total=%d",
            departOrder, arriveOrder, sm.TotalStops)
    }
    if departOrder >= arriveOrder {
        return 0, fmt.Errorf("出发站序必须小于到达站序: depart=%d, arrive=%d",
            departOrder, arriveOrder)
    }

    segmentCount := arriveOrder - departOrder
    return uint64((1 << segmentCount) - 1) << departOrder, nil
}

// SeatBitmap 座位位图管理器
type SeatBitmap struct {
    Status uint64
}

// NewSeatBitmap 创建座位位图
func NewSeatBitmap(status uint64) *SeatBitmap {
    return &SeatBitmap{Status: status}
}

// IsAvailable 检查指定区间是否可售
func (sb *SeatBitmap) IsAvailable(mask uint64) bool {
    return (sb.Status & mask) == 0
}

// Book 购票（占用区间）
func (sb *SeatBitmap) Book(mask uint64) error {
    if !sb.IsAvailable(mask) {
        return fmt.Errorf("区间已被占用")
    }
    sb.Status |= mask
    return nil
}

// Refund 退票（释放区间）
func (sb *SeatBitmap) Refund(mask uint64) {
    sb.Status &^= mask
}

// GetStatus 获取当前状态
func (sb *SeatBitmap) GetStatus() uint64 {
    return sb.Status
}

// OccupiedSegments 获取已占用的区间列表
func (sb *SeatBitmap) OccupiedSegments(totalSegments int) []int {
    var segments []int
    for i := 0; i < totalSegments; i++ {
        if sb.Status&(1<<i) != 0 {
            segments = append(segments, i)
        }
    }
    return segments
}

// AvailableRanges 获取可售的连续区间范围
// 返回: [][2]int, 每个元素是 [起始区间, 结束区间)
func (sb *SeatBitmap) AvailableRanges(totalSegments int) [][2]int {
    var ranges [][2]int
    start := -1

    for i := 0; i < totalSegments; i++ {
        isOccupied := sb.Status&(1<<i) != 0

        if !isOccupied && start == -1 {
            start = i
        } else if isOccupied && start != -1 {
            ranges = append(ranges, [2]int{start, i})
            start = -1
        }
    }

    if start != -1 {
        ranges = append(ranges, [2]int{start, totalSegments})
    }

    return ranges
}
```

### 7.3 选座算法完整代码 (algorithm/seat_select.go)

```go
package algorithm

import (
    "sort"
)

// Seat 座位信息
type Seat struct {
    ID         int64
    CarriageID int64
    SeatNo     string
    Row        int
    Column     string
    SeatType   string
    Position   string // window/aisle/middle
    Status     uint64 // 当前位图状态
}

// SeatLayout 座位布局
type SeatLayout struct {
    SeatType   string
    Columns    []string
    RowCount   int
    WindowCols []string
    AisleCols  []string
    MiddleCols []string
    GroupLeft  []string
    GroupRight []string
}

// 预定义布局
var Layouts = map[string]SeatLayout{
    "二等座": {
        SeatType:   "二等座",
        Columns:    []string{"A", "B", "C", "D", "F"},
        RowCount:   18,
        WindowCols: []string{"A", "F"},
        AisleCols:  []string{"C", "D"},
        MiddleCols: []string{"B"},
        GroupLeft:  []string{"A", "B", "C"},
        GroupRight: []string{"D", "F"},
    },
    "一等座": {
        SeatType:   "一等座",
        Columns:    []string{"A", "C", "D", "F"},
        RowCount:   13,
        WindowCols: []string{"A", "F"},
        AisleCols:  []string{"C", "D"},
        MiddleCols: []string{},
        GroupLeft:  []string{"A", "C"},
        GroupRight: []string{"D", "F"},
    },
    "商务座": {
        SeatType:   "商务座",
        Columns:    []string{"A", "C", "F"},
        RowCount:   8,
        WindowCols: []string{"A", "F"},
        AisleCols:  []string{"C"},
        MiddleCols: []string{},
        GroupLeft:  []string{"A", "C"},
        GroupRight: []string{"F"},
    },
}

// SeatPreference 选座偏好
type SeatPreference struct {
    PreferWindow bool
    PreferAisle  bool
    SpecificSeat string
}

// SelectResult 选座结果
type SelectResult struct {
    Seats    []Seat
    SameRow  bool
    Adjacent bool
}

// SeatSelector 选座器
type SeatSelector struct {
    layout   SeatLayout
    colOrder map[string]int
}

// NewSeatSelector 创建选座器
func NewSeatSelector(seatType string) *SeatSelector {
    layout, ok := Layouts[seatType]
    if !ok {
        layout = Layouts["二等座"]
    }

    colOrder := map[string]int{"A": 0, "B": 1, "C": 2, "D": 3, "F": 4}

    return &SeatSelector{
        layout:   layout,
        colOrder: colOrder,
    }
}

// SelectSingle 单人选座
func (ss *SeatSelector) SelectSingle(seats []Seat, pref SeatPreference, mask uint64) *Seat {
    // 筛选可售座位
    available := ss.filterAvailable(seats, mask)
    if len(available) == 0 {
        return nil
    }

    // 指定座位
    if pref.SpecificSeat != "" {
        for i := range available {
            if available[i].SeatNo == pref.SpecificSeat {
                return &available[i]
            }
        }
        return nil
    }

    // 按偏好筛选
    var preferred []Seat
    for _, seat := range available {
        if pref.PreferWindow && seat.Position == "window" {
            preferred = append(preferred, seat)
        } else if pref.PreferAisle && seat.Position == "aisle" {
            preferred = append(preferred, seat)
        }
    }

    if len(preferred) > 0 {
        ss.sortByRow(preferred)
        return &preferred[0]
    }

    ss.sortByRow(available)
    return &available[0]
}

// SelectMultiple 多人选座
func (ss *SeatSelector) SelectMultiple(seats []Seat, count int, mask uint64) *SelectResult {
    available := ss.filterAvailable(seats, mask)
    if len(available) < count {
        return nil
    }

    rowSeats := ss.groupByRow(available)

    // 策略1: 同排连座
    if result := ss.findSameRowConsecutive(rowSeats, count); result != nil {
        return result
    }

    // 策略2: 相邻排
    if result := ss.findAdjacentRows(rowSeats, count); result != nil {
        return result
    }

    // 策略3: 分散分配
    return ss.findAny(available, count)
}

func (ss *SeatSelector) filterAvailable(seats []Seat, mask uint64) []Seat {
    var result []Seat
    for _, seat := range seats {
        if (seat.Status & mask) == 0 {
            result = append(result, seat)
        }
    }
    return result
}

func (ss *SeatSelector) groupByRow(seats []Seat) map[int][]Seat {
    result := make(map[int][]Seat)
    for _, seat := range seats {
        result[seat.Row] = append(result[seat.Row], seat)
    }
    return result
}

func (ss *SeatSelector) findSameRowConsecutive(rowSeats map[int][]Seat, count int) *SelectResult {
    for row := 1; row <= ss.layout.RowCount; row++ {
        seats, ok := rowSeats[row]
        if !ok || len(seats) < count {
            continue
        }

        // 左侧组
        left := ss.filterByColumns(seats, ss.layout.GroupLeft)
        if cons := ss.findConsecutive(left, count); cons != nil {
            return &SelectResult{Seats: cons, SameRow: true, Adjacent: true}
        }

        // 右侧组
        right := ss.filterByColumns(seats, ss.layout.GroupRight)
        if cons := ss.findConsecutive(right, count); cons != nil {
            return &SelectResult{Seats: cons, SameRow: true, Adjacent: true}
        }

        // C+D
        if count == 2 {
            var c, d *Seat
            for i := range seats {
                if seats[i].Column == "C" {
                    c = &seats[i]
                }
                if seats[i].Column == "D" {
                    d = &seats[i]
                }
            }
            if c != nil && d != nil {
                return &SelectResult{Seats: []Seat{*c, *d}, SameRow: true, Adjacent: true}
            }
        }
    }
    return nil
}

func (ss *SeatSelector) findAdjacentRows(rowSeats map[int][]Seat, count int) *SelectResult {
    for split := count - 1; split >= (count+1)/2; split-- {
        remain := count - split

        for row := 1; row < ss.layout.RowCount; row++ {
            s1, ok1 := rowSeats[row]
            s2, ok2 := rowSeats[row+1]
            if !ok1 || !ok2 {
                continue
            }

            left1 := ss.filterByColumns(s1, ss.layout.GroupLeft)
            g1 := ss.findConsecutive(left1, split)
            usedLeft := g1 != nil

            if g1 == nil {
                right1 := ss.filterByColumns(s1, ss.layout.GroupRight)
                g1 = ss.findConsecutive(right1, split)
            }

            if g1 == nil {
                continue
            }

            var targetCols []string
            if usedLeft {
                targetCols = ss.layout.GroupLeft
            } else {
                targetCols = ss.layout.GroupRight
            }

            target2 := ss.filterByColumns(s2, targetCols)
            g2 := ss.findConsecutive(target2, remain)

            if g2 == nil {
                if usedLeft {
                    targetCols = ss.layout.GroupRight
                } else {
                    targetCols = ss.layout.GroupLeft
                }
                target2 = ss.filterByColumns(s2, targetCols)
                g2 = ss.findConsecutive(target2, remain)
            }

            if g2 != nil {
                return &SelectResult{
                    Seats:    append(g1, g2...),
                    SameRow:  false,
                    Adjacent: true,
                }
            }
        }
    }
    return nil
}

func (ss *SeatSelector) findAny(seats []Seat, count int) *SelectResult {
    if len(seats) < count {
        return nil
    }
    ss.sortByRow(seats)
    return &SelectResult{
        Seats:    seats[:count],
        SameRow:  false,
        Adjacent: false,
    }
}

func (ss *SeatSelector) filterByColumns(seats []Seat, cols []string) []Seat {
    colSet := make(map[string]bool)
    for _, c := range cols {
        colSet[c] = true
    }
    var result []Seat
    for _, s := range seats {
        if colSet[s.Column] {
            result = append(result, s)
        }
    }
    return result
}

func (ss *SeatSelector) findConsecutive(seats []Seat, count int) []Seat {
    if len(seats) < count {
        return nil
    }

    ss.sortByColumn(seats)

    for i := 0; i <= len(seats)-count; i++ {
        ok := true
        for j := 1; j < count; j++ {
            p := ss.colOrder[seats[i+j-1].Column]
            c := ss.colOrder[seats[i+j].Column]
            if c-p != 1 {
                ok = false
                break
            }
        }
        if ok {
            result := make([]Seat, count)
            copy(result, seats[i:i+count])
            return result
        }
    }
    return nil
}

func (ss *SeatSelector) sortByRow(seats []Seat) {
    sort.Slice(seats, func(i, j int) bool {
        if seats[i].Row != seats[j].Row {
            return seats[i].Row < seats[j].Row
        }
        return ss.colOrder[seats[i].Column] < ss.colOrder[seats[j].Column]
    })
}

func (ss *SeatSelector) sortByColumn(seats []Seat) {
    sort.Slice(seats, func(i, j int) bool {
        return ss.colOrder[seats[i].Column] < ss.colOrder[seats[j].Column]
    })
}
```

---

## 附录

### A. 常用位运算速查

| 操作 | 代码 | 说明 |
|-----|------|------|
| 设置某位为1 | `n \| (1 << k)` | 将第k位设为1 |
| 设置某位为0 | `n & ^(1 << k)` | 将第k位设为0 |
| 翻转某位 | `n ^ (1 << k)` | 翻转第k位 |
| 检查某位 | `(n >> k) & 1` | 获取第k位的值 |
| 生成n个1 | `(1 << n) - 1` | 低n位全为1 |
| 位与 | `a & b` | 两个都为1才为1 |
| 位或 | `a \| b` | 有一个为1就为1 |
| 位异或 | `a ^ b` | 不同为1，相同为0 |
| 位清除 | `a &^ b` | 清除a中b为1的位 |

### B. 参考资料

1. 12306 技术架构分析
2. Redis 位图操作文档
3. 分布式锁实现最佳实践
4. 高并发系统设计模式

---

> 文档版本: 1.0
>
> 整理日期: 2024年
>
> 本文档基于对话内容整理，包含购票系统的核心设计思路和算法实现，可作为学习参考。