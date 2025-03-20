# 访客系统概要设计文档

## 1. 数据结构设计

### 1.1 访客信息表(visitor)
```sql
CREATE TABLE visitor (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '访客ID',
    name VARCHAR(50) NOT NULL COMMENT '访客姓名',
    phone VARCHAR(20) NOT NULL COMMENT '手机号',
    id_card VARCHAR(18) NOT NULL COMMENT '身份证号',
    purpose VARCHAR(200) NOT NULL COMMENT '来访目的',
    car_number VARCHAR(20) COMMENT '车牌号',
    status TINYINT NOT NULL DEFAULT 0 COMMENT '状态(0-待审批 1-已通过 2-已拒绝)',
    visit_code VARCHAR(32) COMMENT '访客码',
    created_time DATETIME NOT NULL COMMENT '创建时间',
    updated_time DATETIME NOT NULL COMMENT '更新时间'
) COMMENT '访客信息表';
```

### 1.2 被访人信息表(employee)
```sql
CREATE TABLE employee (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '员工ID',
    name VARCHAR(50) NOT NULL COMMENT '员工姓名',
    department VARCHAR(50) NOT NULL COMMENT '所属部门',
    phone VARCHAR(20) NOT NULL COMMENT '联系电话',
    status TINYINT NOT NULL DEFAULT 1 COMMENT '状态(0-禁用 1-启用)',
    created_time DATETIME NOT NULL COMMENT '创建时间',
    updated_time DATETIME NOT NULL COMMENT '更新时间'
) COMMENT '被访人信息表';
```

### 1.3 访问记录表(visit_record)
```sql
CREATE TABLE visit_record (
    id BIGINT PRIMARY KEY AUTO_INCREMENT COMMENT '记录ID',
    visitor_id BIGINT NOT NULL COMMENT '访客ID',
    employee_id BIGINT NOT NULL COMMENT '被访人ID',
    visit_time DATETIME COMMENT '来访时间',
    leave_time DATETIME COMMENT '离开时间',
    status TINYINT NOT NULL COMMENT '状态(0-待审批 1-已通过 2-已拒绝 3-已完成)',
    created_time DATETIME NOT NULL COMMENT '创建时间',
    updated_time DATETIME NOT NULL COMMENT '更新时间',
    FOREIGN KEY (visitor_id) REFERENCES visitor(id),
    FOREIGN KEY (employee_id) REFERENCES employee(id)
) COMMENT '访问记录表';
```

## 2. 实体关系图

```mermaid
erDiagram
    visitor ||--o{ visit_record : has
    employee ||--o{ visit_record : has
    
    visitor {
        bigint id PK
        string name
        string phone
        string id_card
        string purpose
        string car_number
        int status
        string visit_code
    }
    
    employee {
        bigint id PK
        string name
        string department
        string phone
        int status
    }
    
    visit_record {
        bigint id PK
        bigint visitor_id FK
        bigint employee_id FK
        datetime visit_time
        datetime leave_time
        int status
    }
```

## 3. 访客登记流程图

```mermaid
sequenceDiagram
    participant V as 访客端
    participant S as 服务端
    participant D as 数据库
    participant E as 被访人
    
    V->>S: 提交访客信息
    S->>D: 保存访客信息
    S->>D: 创建访问记录
    S->>E: 发送审批通知
    E->>S: 审批处理
    alt 审批通过
        S->>D: 更新访问状态
        S->>V: 生成访客码
    else 审批拒绝
        S->>D: 更新访问状态
        S->>V: 返回拒绝原因
    end
```

## 4. 核心接口设计

### 4.1 访客登记接口
```
POST /api/visitor/register

Request:
{
    "name": "string",      // 访客姓名
    "phone": "string",     // 手机号
    "idCard": "string",    // 身份证号
    "purpose": "string",   // 来访目的
    "carNumber": "string", // 车牌号
    "employeeId": "long"   // 被访人ID
}

Response:
{
    "code": 200,
    "message": "success",
    "data": {
        "visitorId": "long",    // 访客ID
        "recordId": "long"      // 访问记录ID
    }
}
```

### 4.2 访客审批接口
```
POST /api/visit/approve

Request:
{
    "recordId": "long",    // 访问记录ID
    "approved": "boolean", // 是否通过
    "visitTime": "string", // 预约时间
    "remark": "string"     // 备注说明
}

Response:
{
    "code": 200,
    "message": "success",
    "data": {
        "visitCode": "string"   // 访客码
    }
}
```