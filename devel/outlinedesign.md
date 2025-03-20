# 小鹏访客系统概要设计文档

## 1. 系统架构

### 1.1 总体架构
```
+----------------+    +----------------+    +----------------+
|   访客端API    |    |   员工端API    |    |   管理端API    |
+----------------+    +----------------+    +----------------+
         |                   |                    |
         +-------------------+--------------------+
                            |
                    +----------------+
                    |  业务服务层     |
                    +----------------+
                            |
                    +----------------+
                    |   数据存储层    |
                    +----------------+
```

## 2. 数据建模

### 2.1 实体关系图
```mermaid
erDiagram
    Visitor ||--o{ Appointment : makes
    Visitor ||--o{ VisitorRecord : has
    Employee ||--o{ Appointment : approves
    Appointment ||--|{ AccessControl : generates
    VisitorRecord ||--|| AccessControl : controls
    
    Visitor {
        int visitor_id
        string name
        string phone
        string id_card
        string face_path
        int status
    }
    
    Appointment {
        int appointment_id
        int visitor_id
        int employee_id
        datetime visit_time
        string purpose
        int status
    }
    
    Employee {
        int employee_id
        string name
        string department
        string phone
        int role
    }
    
    VisitorRecord {
        int record_id
        int visitor_id
        datetime check_in
        datetime check_out
        string location
    }
    
    AccessControl {
        int access_id
        int appointment_id
        string auth_code
        datetime valid_from
        datetime valid_to
    }
```

### 2.2 数据库表结构
[核心表结构设计]

## 3. 关键流程

### 3.1 访客预约流程
```mermaid
sequenceDiagram
    participant V as 访客
    participant S as 系统
    participant E as 员工
    participant A as 门禁
    
    V->>S: 提交预约申请
    S->>E: 发送审批通知
    E->>S: 审批处理
    S->>V: 返回预约结果
    S->>A: 生成门禁授权
```

### 3.2 访客登记流程
```mermaid
sequenceDiagram
    participant V as 访客
    participant K as 登记终端
    participant S as 系统
    participant A as 门禁系统
    
    V->>K: 扫描身份证
    K->>S: 验证预约信息
    K->>K: 采集人脸
    K->>S: 提交登记信息
    S->>A: 激活门禁权限
    S->>K: 打印访客证
```

## 4. 接口设计

### 4.1 API接口列表
- /api/visitor/appointment
- /api/visitor/register
- /api/employee/approve
- /api/access/verify
[详细接口文档]

## 5. 安全设计

### 5.1 数据安全
- 传输加密：TLS 1.2
- 存储加密：AES-256
- 访问控制：RBAC模型

### 5.2 身份认证
- JWT令牌
- 生物特征认证
- 动态授权码

## 6. 部署架构

### 6.1 系统部署图
[部署架构图]