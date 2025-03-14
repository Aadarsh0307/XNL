-- Users Table (Holds customer details, indexed by user_id)
CREATE TABLE users (
    user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    full_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone_number VARCHAR(20) UNIQUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Accounts Table (Holds financial accounts per user)
CREATE TABLE accounts (
    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id) ON DELETE CASCADE,
    account_type VARCHAR(50) CHECK (account_type IN ('savings', 'checking', 'investment')),
    balance DECIMAL(18,2) NOT NULL DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Transactions Table (Handles financial transactions efficiently)
CREATE TABLE transactions (
    transaction_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID REFERENCES accounts(account_id) ON DELETE CASCADE,
    transaction_type VARCHAR(50) CHECK (transaction_type IN ('deposit', 'withdrawal', 'transfer', 'purchase')),
    amount DECIMAL(18,2) NOT NULL,
    currency VARCHAR(10) NOT NULL DEFAULT 'USD',
    status VARCHAR(50) CHECK (status IN ('pending', 'completed', 'failed')),
    created_at TIMESTAMP DEFAULT NOW()
) PARTITION BY RANGE (created_at);

-- Transaction Partitioning (Partitioning by month for fast querying)
CREATE TABLE transactions_2025_01 PARTITION OF transactions
    FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');
CREATE TABLE transactions_2025_02 PARTITION OF transactions
    FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');

-- Order Book Table (For market orders in trading scenario)
CREATE TABLE order_book (
    order_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(user_id),
    asset VARCHAR(50) NOT NULL,
    order_type VARCHAR(10) CHECK (order_type IN ('buy', 'sell')),
    quantity DECIMAL(18,6) NOT NULL,
    price DECIMAL(18,2) NOT NULL,
    status VARCHAR(20) CHECK (status IN ('open', 'filled', 'cancelled')),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Indexing Strategy for Optimized Query Performance
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_transactions_account ON transactions(account_id);
CREATE INDEX idx_transactions_time ON transactions(created_at DESC);
CREATE INDEX idx_order_book_status ON order_book(status);

-- Sharding Strategy: Separate databases for transaction-heavy and analytical workloads
-- Implement consistent hashing across database instances for high-availability

-- Analytics Table (Materialized view for real-time insights)
CREATE MATERIALIZED VIEW transaction_analytics AS
SELECT account_id, transaction_type, SUM(amount) AS total_amount, COUNT(*) AS transaction_count
FROM transactions
GROUP BY account_id, transaction_type;

-- Security & Compliance (Role-based access control & encryption)
ALTER TABLE users ADD COLUMN password_hash TEXT;
GRANT SELECT, INSERT, UPDATE ON users TO fintech_readonly_role;
