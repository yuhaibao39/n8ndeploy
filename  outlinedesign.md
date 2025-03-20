# B端官网系统概要设计文档 V4

## 1. 数据建模

### 1.1 核心实体
```sql
-- 产品表
Product {
  id: int(pk)
  name: varchar
  description: text
  category_id: int(fk) 
  features: text
  tech_specs: text
  image_urls: json
  sort: int
  create_time: datetime
  status: tinyint
}

-- 解决方案表  
Solution {
  id: int(pk)
  title: varchar
  content: text
  highlights: json
  arch_diagram: varchar
  industry_id: int(fk)
  sort: int
  create_time: datetime
  status: tinyint
}

-- 行业表
Industry {
  id: int(pk)  
  name: varchar
  icon: varchar
  sort: int
  status: tinyint
}

-- 客户案例表
Case {
  id: int(pk)
  title: varchar
  description: text
  industry_id: int(fk)
  solution_id: int(fk) 
  logo: varchar
  images: json
  sort: int
  create_time: datetime
  status: tinyint
}

-- 类别表
Category {
  id: int(pk)
  name: varchar
  icon: varchar
  parent_id: int
  sort: int 
  status: tinyint
}
```

## 2. 实体关系图
```mermaid
erDiagram
    Category ||--o{ Product : contains
    Industry ||--o{ Solution : belongs
    Industry ||--o{ Case : belongs  
    Solution ||--o{ Case : includes
```

## 3. 核心流程

### 3.1 首页加载流程
```mermaid
sequenceDiagram
    participant C as Client
    participant F as Frontend  
    participant B as Backend
    participant D as Database

    C->>F: 访问首页
    F->>B: 请求首页数据
    B->>D: 查询数据库
    D-->>B: 返回数据
    B-->>F: 返回首页数据
    F-->>C: 渲染页面
```

### 3.2 方案筛选流程
```mermaid
sequenceDiagram
    participant C as Client
    participant F as Frontend
    participant B as Backend
    participant D as Database

    C->>F: 选择行业
    F->>B: 请求对应方案
    B->>D: 条件查询 
    D-->>B: 返回方案数据
    B-->>F: 返回筛选结果
    F-->>C: 更新页面
```

### 3.3 案例查看流程
```mermaid
sequenceDiagram
    participant C as Client
    participant F as Frontend
    participant B as Backend
    participant D as Database

    C->>F: 查看案例详情
    F->>B: 获取案例信息
    B->>D: 查询案例
    D-->>B: 返回案例数据
    B->>D: 关联查询方案
    D-->>B: 返回关联数据
    B-->>F: 返回完整信息
    F-->>C: 展示案例详情
```