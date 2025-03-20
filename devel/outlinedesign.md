# 访客系统概要设计文档

## 1. 数据结构设计

### 1.1 访客信息表(visitor)
```sql
CREATE TABLE `visitor` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `name` varchar(50) NOT NULL COMMENT '访客姓名',
  `phone` varchar(20) NOT NULL COMMENT '手机号码',
  `id_card` varchar(18) NOT NULL COMMENT '身份证号',
  `visit_reason` varchar(200) NOT NULL COMMENT '来访事由',
  `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '状态(0-待审核 1-已通过 2-已拒绝)',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `update_time` datetime NOT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_phone` (`phone`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='访客信息表';
```

### 1.2 访问记录表(visit_record) 
```sql
CREATE TABLE `visit_record` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `visitor_id` bigint(20) NOT NULL COMMENT '访客ID',
  `visit_time` datetime NOT NULL COMMENT '来访时间',
  `leave_time` datetime DEFAULT NULL COMMENT '离开时间',
  `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '状态(0-进行中 1-已结束)',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `update_time` datetime NOT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_visitor_id` (`visitor_id`),
  CONSTRAINT `fk_visit_record_visitor` FOREIGN KEY (`visitor_id`) REFERENCES `visitor` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='访问记录表';
```

## 2. 实体关系图

```mermaid
erDiagram
    VISITOR ||--o{ VISIT_RECORD : has
    VISITOR {
        bigint id PK
        string name
        string phone
        string id_card
        string visit_reason
        int status
        datetime create_time
        datetime update_time
    }
    VISIT_RECORD {
        bigint id PK
        bigint visitor_id FK
        datetime visit_time
        datetime leave_time
        int status
        datetime create_time
        datetime update_time
    }
```

## 3. 核心流程时序图

```mermaid
sequenceDiagram
    participant C as 前端
    participant S as 后端服务
    participant DB as 数据库
    
    C->>S: 1.提交访客信息
    S->>DB: 2.保存访客记录
    DB-->>S: 3.返回结果
    S-->>C: 4.返回访客ID
    
    C->>S: 5.访客到访登记
    S->>DB: 6.创建访问记录
    DB-->>S: 7.返回结果
    S-->>C: 8.返回登记结果
    
    C->>S: 9.访客离开登记
    S->>DB: 10.更新访问记录
    DB-->>S: 11.返回结果 
    S-->>C: 12.返回登记结果
```

## 4. 关键接口设计

### 4.1 访客登记接口
```
POST /api/visitor/register

Request:
{
    "name": "张三",
    "phone": "13800138000",
    "idCard": "110101199001011234",
    "visitReason": "商务洽谈"
}

Response:
{
    "code": 200,
    "message": "success",
    "data": {
        "visitorId": 123456
    }
}
```

### 4.2 访客到访登记接口
```
POST /api/visit/checkin

Request:
{
    "visitorId": 123456,
    "visitTime": "2024-01-01 10:00:00"
}

Response:
{
    "code": 200,
    "message": "success",
    "data": {
        "recordId": 789012
    }
}
```

### 4.3 访客离开登记接口
```
POST /api/visit/checkout

Request:
{
    "recordId": 789012,
    "leaveTime": "2024-01-01 11:30:00"
}

Response:
{
    "code": 200,
    "message": "success"
}
```