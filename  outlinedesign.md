# 小鹏访客系统概要设计文档

## 1. 数据结构设计

### 1.1 核心数据表

#### 访客信息表(visitor)
```sql
CREATE TABLE visitor (
    id BIGINT PRIMARY KEY,           -- 访客ID
    name VARCHAR(50),                 -- 姓名
    id_card VARCHAR(18),             -- 身份证号
    phone VARCHAR(11),               -- 手机号
    photo_url VARCHAR(200),          -- 照片URL
    created_at DATETIME,             -- 创建时间
    updated_at DATETIME              -- 更新时间
);
```

#### 预约记录表(appointment)
```sql
CREATE TABLE appointment (
    id BIGINT PRIMARY KEY,           -- 预约ID 
    visitor_id BIGINT,               -- 访客ID
    employee_id BIGINT,              -- 被访人ID
    visit_date DATE,                 -- 访问日期
    visit_start TIME,                -- 开始时间
    visit_end TIME,                  -- 结束时间
    purpose VARCHAR(200),            -- 来访目的
    status TINYINT,                  -- 状态(0待审批/1已通过/2已拒绝)
    appointment_code VARCHAR(32),     -- 预约码
    created_at DATETIME              -- 创建时间
);
```

#### 访问记录表(visit_record)
```sql
CREATE TABLE visit_record (
    id BIGINT PRIMARY KEY,           -- 记录ID
    appointment_id BIGINT,           -- 预约ID
    check_in_time DATETIME,          -- 签到时间
    check_out_time DATETIME,         -- 签出时间
    card_no VARCHAR(32),             -- 访客证号
    status TINYINT,                  -- 状态(1在访/2已结束)
    created_at DATETIME              -- 创建时间
);
```

## 2. 实体关系图

```mermaid
erDiagram
    visitor ||--o{ appointment : creates
    employee ||--o{ appointment : receives
    appointment ||--o| visit_record : generates
    
    visitor {
        bigint id
        string name
        string id_card
        string phone
        string photo_url
    }
    
    appointment {
        bigint id
        bigint visitor_id
        bigint employee_id
        date visit_date
        time visit_start
        time visit_end
        string purpose
        int status
    }
    
    visit_record {
        bigint id
        bigint appointment_id
        datetime check_in_time
        datetime check_out_time
        string card_no
        int status
    }
```

## 3. 核心流程时序图

### 3.1 访客预约流程

```mermaid
sequenceDiagram
    participant V as 访客端
    participant S as 服务端
    participant E as 员工端
    
    V->>S: 提交访客信息
    S-->>V: 返回填写结果
    V->>S: 发起预约申请
    S->>E: 推送预约通知
    E->>S: 审批处理
    S-->>V: 返回预约结果
    alt 审批通过
        S->>V: 生成预约码
    else 审批拒绝
        S->>V: 发送拒绝通知
    end
```

### 3.2 访客接待流程

```mermaid
sequenceDiagram
    participant V as 访客
    participant F as 前台
    participant S as 服务端
    
    V->>F: 出示预约码
    F->>S: 扫码验证
    S-->>F: 返回预约信息
    F->>S: 录入到访信息
    S-->>F: 生成访客证
    F->>V: 发放访客证
    
    V->>F: 归还访客证
    F->>S: 登记离访
    S-->>F: 更新访问状态
```

## 4. 关键接口设计

### 4.1 预约创建接口
```json
POST /api/appointment/create
Request:
{
    "visitor": {
        "name": "张三",
        "idCard": "330xxxxxxxxxxxxxxx",
        "phone": "138xxxxxxxx"
    },
    "appointment": {
        "employeeId": 12345,
        "visitDate": "2024-01-20",
        "visitStart": "09:00",
        "visitEnd": "18:00",
        "purpose": "业务洽谈"
    }
}

Response:
{
    "code": 200,
    "data": {
        "appointmentId": 67890,
        "appointmentCode": "AP202401200001"
    }
}
```

### 4.2 预约审批接口
```json
POST /api/appointment/approve
Request:
{
    "appointmentId": 67890,
    "approved": true,
    "comment": "同意访问"
}

Response:
{
    "code": 200,
    "message": "审批成功"
}
```