### 数据库设计方案：TDE (二次元交易所)
---
## 一、用户相关表

### 1. 用户基础表（users）

```sql
CREATE TABLE users (

  user_id SERIAL PRIMARY KEY,

  username CHAR(20) NOT NULL,

  email VARCHAR(20) NOT NULL UNIQUE,

  password_hash VARCHAR(255) NOT NULL,

  avatar_url VARCHAR(255),

  registration_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

  last_login TIMESTAMP,

  status VARCHAR(20) DEFAULT 'active',

  is_verified BOOLEAN DEFAULT false,

  bankruptcy_count INT DEFAULT 0,

  last_bankruptcy_date TIMESTAMP,

  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP

);

```

### 2. 用户安全验证表（user_security）

```sql
CREATE TABLE user_security (

  security_id SERIAL PRIMARY KEY,

  user_id INT NOT NULL,

  two_factor_enabled BOOLEAN DEFAULT false,

  two_factor_secret CHAR(32),

  reset_password_token CHAR(32),

);
```

### 3. 用户设置表（user_settings）

```sql
CREATE TABLE user_settings (

  setting_id SERIAL PRIMARY KEY,

  user_id INT NOT NULL,

  language VARCHAR(10) DEFAULT 'zh-CN',

  theme VARCHAR(10) DEFAULT 'light',

  chart_settings JSONB,

  notification_settings JSONB

);
```

### 4. 资金卡表（fund_cards）

```sql
CREATE TABLE fund_cards (

  card_number BIGINT PRIMARY KEY,

  user_id INT NOT NULL,

  balance DECIMAL(40,8) DEFAULT 0,

  status CHAR(20) DEFAULT 'active'

);
```

### 5. 金融卡表（finance_cards）

```sql
CREATE TABLE finance_cards (

  card_number BIGINT PRIMARY KEY,

  user_id INT NOT NULL,

  balance DECIMAL(40,8) DEFAULT 0,

  status VARCHAR(20) DEFAULT 'active'

);
```

## 二、管理员相关表

### 1. 管理员表（admins）

```sql
CREATE TABLE admins (

  admin_id SERIAL PRIMARY KEY,

  username VARCHAR(50) NOT NULL UNIQUE,

  password_hash VARCHAR(255) NOT NULL,

  role_id INT NOT NULL,

  status VARCHAR(20) DEFAULT 'active',

  last_login TIMESTAMP

);
```

### 2. 管理员角色表（admin_roles）

```sql
CREATE TABLE admin_roles (

  role_id SERIAL PRIMARY KEY,

  role_name VARCHAR(50) NOT NULL UNIQUE,

  description TEXT,

  permissions JSONB NOT NULL

);
```

### 3. 多重签名表（multi_signatures）

```sql
CREATE TABLE multi_signatures (

  signature_id SERIAL PRIMARY KEY,

  operation_type VARCHAR(50) NOT NULL,

  operation_data JSONB NOT NULL,

  required_signatures INT NOT NULL,

  current_signatures INT DEFAULT 0,

  status VARCHAR(20) DEFAULT 'pending',

  initiator_admin_id INT NOT NULL,

  signed_by JSONB,

  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP

);
```

## 三、交易相关表

### 1. 股票表（stocks）

```sql
CREATE TABLE stocks (

  stock_id SERIAL PRIMARY KEY,

  symbol VARCHAR(20) NOT NULL UNIQUE,

  name VARCHAR(100) NOT NULL,

  description TEXT,

  team_etf_id INT,

  sector_id INT,

  issue_date TIMESTAMP,

  issue_price DECIMAL(40, 8),

  max_supply DECIMAL(40, 8),

  circulating_supply DECIMAL(40, 8),

  current_price DECIMAL(40, 8),

  highest_price DECIMAL(40, 8),

  lowest_price DECIMAL(40, 8),

  popularity_rank INT,

  market_cap_rank INT,

  official_link VARCHAR(255),

  social_media JSONB,

  status VARCHAR(20) DEFAULT 'active'

);
```

### 2. 团队ETF表（team_etfs）

```sql
CREATE TABLE team_etfs (

  etf_id SERIAL PRIMARY KEY,

  symbol VARCHAR(20) NOT NULL UNIQUE,

  name VARCHAR(100) NOT NULL,

  description TEXT,

  sector_id INT,

  components JSONB, *-- 包含的股票及权重*

  current_price DECIMAL(40, 8),

  status VARCHAR(20) DEFAULT 'active'

);
```

### 3. 板块表（sectors）

```sql
CREATE TABLE sectors (

  sector_id SERIAL PRIMARY KEY,

  name VARCHAR(100) NOT NULL UNIQUE,

  description TEXT,

  current_index DECIMAL(40, 8),

  status VARCHAR(20) DEFAULT 'active'

);
```

### 4. 交易记录表（trades）

```sql
CREATE TABLE trades (

  trade_id SERIAL PRIMARY KEY,

  user_id INT NOT NULL,

  trade_type VARCHAR(20) NOT NULL, *-- 'buy', 'sell'*

  asset_type VARCHAR(20) NOT NULL, *-- 'stock', 'team_etf', 'sector_etf'*

  asset_id INT NOT NULL,

  quantity DECIMAL(20, 8) NOT NULL,

  price DECIMAL(40, 8) NOT NULL,

  total_amount DECIMAL(40, 8) NOT NULL,

  fee DECIMAL(40, 8) NOT NULL,

  leverage INT DEFAULT 1,

  trade_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

  status VARCHAR(20) DEFAULT 'completed'

);
```

### 5. 用户持仓表（user_holdings）

```sql
CREATE TABLE user_holdings (

  holding_id SERIAL PRIMARY KEY,

  user_id INT NOT NULL,

  asset_type VARCHAR(20) NOT NULL, *-- 'stock', 'team_etf', 'sector_etf'*

  asset_id INT NOT NULL,

  quantity DECIMAL(20, 8) NOT NULL,

  average_cost DECIMAL(20, 8) NOT NULL,

  current_value DECIMAL(20, 8) NOT NULL,

  profit_loss DECIMAL(20, 8) NOT NULL

);
```

### 6. 行情数据表（price_history）

```sql
CREATE TABLE price_history (

  history_id SERIAL PRIMARY KEY,

  asset_type VARCHAR(20) NOT NULL,

  asset_id INT NOT NULL,

  price_time TIMESTAMP NOT NULL,

  open_price DECIMAL(20, 8) NOT NULL,

  high_price DECIMAL(20, 8) NOT NULL,

  low_price DECIMAL(20, 8) NOT NULL,

  close_price DECIMAL(20, 8) NOT NULL,

  volume DECIMAL(20, 8) NOT NULL,

  timeframe VARCHAR(10) NOT NULL *-- '1m', '5m', '15m', '1h', '4h', '1d'*

);
```

### 7. 合约表（contracts）

```sql
CREATE TABLE contracts (

  contract_id SERIAL PRIMARY KEY,

  user_id INT NOT NULL,

  contract_type VARCHAR(20) NOT NULL, *-- 'perpetual', 'options', 'event'*

  direction VARCHAR(10) NOT NULL, *-- 'long', 'short'*

  asset_type VARCHAR(20) NOT NULL,

  asset_id INT NOT NULL,

  open_price DECIMAL(40, 8) NOT NULL,

  quantity DECIMAL(40, 8) NOT NULL,

  leverage INT NOT NULL,

  liquidation_price DECIMAL(40, 8),

  take_profit DECIMAL(40, 8),

  stop_loss DECIMAL(40, 8),

  open_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

  close_time TIMESTAMP,

  close_price DECIMAL(40, 8),

  profit_loss DECIMAL(40, 8),

  status VARCHAR(20) DEFAULT 'open'

);
```

## 四、其他功能表

### 1. 理财产品表（financial_products）

```sql
CREATE TABLE financial_products (

  product_id SERIAL PRIMARY KEY,

  name VARCHAR(100) NOT NULL,

  description TEXT,

  type VARCHAR(20) NOT NULL, *-- 'current', 'fixed'*

  duration INT, *-- 天数，活期为NULL*

  annual_rate DECIMAL(10, 4) NOT NULL,

  min_amount DECIMAL(20, 8) NOT NULL,

  max_amount DECIMAL(20, 8),

  status VARCHAR(20) DEFAULT 'active'

);
```

### 2. 用户理财表（user_financial_products）

```sql
CREATE TABLE user_financial_products (

  investment_id SERIAL PRIMARY KEY,

  user_id INT NOT NULL,

  product_id INT NOT NULL,

  amount DECIMAL(20, 8) NOT NULL,

  start_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

  end_date TIMESTAMP,

  expected_interest DECIMAL(20, 8) NOT NULL,

  status VARCHAR(20) DEFAULT 'active' *-- 'active', 'redeemed', 'completed'*

);
```

### 3. 交易机器人表（trading_bots）

```sql
CREATE TABLE trading_bots (

  bot_id SERIAL PRIMARY KEY,

  creator_id INT NOT NULL,

  creator_type VARCHAR(10) NOT NULL, *-- 'user', 'admin'*

  name VARCHAR(100) NOT NULL,

  description TEXT,

  strategy JSONB NOT NULL,

  parameters JSONB,

  is_public BOOLEAN DEFAULT false,

  status VARCHAR(20) DEFAULT 'active'

);
```

### 4. 用户机器人表（user_bots）

```sql
CREATE TABLE user_bots (

  user_bot_id SERIAL PRIMARY KEY,

  user_id INT NOT NULL,

  bot_id INT NOT NULL,

  parameters JSONB,

  allocated_funds DECIMAL(20, 8) NOT NULL,

  status VARCHAR(20) DEFAULT 'active',

  profit_loss DECIMAL(20, 8) DEFAULT 0

);
```

### 5. 空投和福利表

```sql
CREATE TABLE airdrops (

  airdrop_id SERIAL PRIMARY KEY,

  name VARCHAR(100) NOT NULL,

  description TEXT,

  asset_type VARCHAR(20) NOT NULL,

  asset_id INT,

  amount DECIMAL(20, 8) NOT NULL,

  conditions JSONB,

  start_time TIMESTAMP NOT NULL,

  end_time TIMESTAMP NOT NULL,

  status VARCHAR(20) DEFAULT 'active'

);
```

```sql
CREATE TABLE user_airdrops (

  user_airdrop_id SERIAL PRIMARY KEY,

  user_id INT NOT NULL,

  airdrop_id INT NOT NULL,

  claimed_amount DECIMAL(20, 8) NOT NULL,

  claimed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP

);
```

```sql
CREATE TABLE benefits (

  benefit_id SERIAL PRIMARY KEY,

  name VARCHAR(100) NOT NULL,

  description TEXT,

  benefit_type VARCHAR(20) NOT NULL,

  value DECIMAL(20, 8) NOT NULL,

  conditions JSONB,

  start_time TIMESTAMP NOT NULL,

  end_time TIMESTAMP NOT NULL,

  max_claims INT,

  current_claims INT DEFAULT 0,

  status VARCHAR(20) DEFAULT 'active'

);
```
