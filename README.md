# sql-cs

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE IF NOT EXISTS uploads (
  id uuid PRIMARY KEY DEFAULT uuid_generate_v4(),
  original_filename text,
  file_path text NOT NULL,
  uploaded_at timestamptz NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_uploads_uploaded_at
ON uploads (uploaded_at DESC);


CREATE TABLE IF NOT EXISTS transactions (
  id uuid PRIMARY KEY DEFAULT uuid_generate_v4(),

  upload_id uuid NOT NULL REFERENCES uploads(id) ON DELETE CASCADE,

  bank text,
  transferred_at timestamptz,         
  amount numeric(12,2),               
  category text,

  raw_ocr jsonb,

  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_tx_transferred_at
ON transactions (transferred_at DESC);

CREATE INDEX IF NOT EXISTS idx_tx_bank
ON transactions (bank);

CREATE INDEX IF NOT EXISTS idx_tx_created_at
ON transactions (created_at DESC);

CREATE INDEX IF NOT EXISTS idx_tx_raw_ocr_gin
ON transactions USING gin (raw_ocr);

CREATE TABLE IF NOT EXISTS users (
  id uuid PRIMARY KEY DEFAULT uuid_generate_v4(),

  username text NOT NULL UNIQUE,
  password_hash text NOT NULL,

  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE INDEX IF NOT EXISTS idx_users_username
ON users (username);

ALTER TABLE uploads
ADD COLUMN user_id uuid REFERENCES users(id) ON DELETE SET NULL;

ALTER TABLE uploads
ADD column created_at timestamptz NOT NULL DEFAULT now();

ALTER TABLE uploads
ADD COLUMN file_hash text;

CREATE UNIQUE INDEX IF NOT EXISTS ux_uploads_file_hash
ON uploads (file_hash);

DO $$ BEGIN
    CREATE TYPE expense_category AS ENUM (
        'ค่าอาหาร/เครื่องดื่ม',
        'ค่าเดินทาง',
        'ค่าของใช้/จิปาถะ',
        'ค่าที่พัก/สาธารณูปโภค',
        'อื่นๆ'
    );
EXCEPTION
    WHEN duplicate_object THEN NULL;
END $$;

ALTER TABLE transactions
ADD COLUMN category_enum expense_category;

-- (optional) ถ้าจะใช้ category_enum แทน category ก็ migrate แล้วค่อย drop category

ALTER TABLE transactions
ADD COLUMN category_source text;  -- เช่น 'ocr_guess' | 'user_selected' | 'missing_memo_user_selected'

ALTER TABLE transactions
ADD COLUMN memo text;

ALTER table users 
ADD COLUMN email text;

select inet_server_addr() as server_addr,
       inet_server_port() as server_port,
       current_database() as db,
       current_user as usr,
       version();

select inet_server_addr(), inet_server_port(), current_database(), current_user;

select count(*) from public.transactions;
select count(*) from public.uploads;

select table_schema, table_name
from information_schema.tables
where table_name = 'transactions';

select table_name
from information_schema.tables
where table_schema = 'public';

select id from transactions order by created_at desc limit 5;

select *
from transactions
where id = 'c8406752-7491-4a82-9c67-396a0235551f';


CREATE TABLE goals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

    user_id UUID NOT NULL,
    
    -- เก็บเดือนเป็นรูปแบบ YYYY-MM เช่น 2026-03
    month VARCHAR(7) NOT NULL,

    -- จำนวนเงิน goal
    amount NUMERIC(12,2) NOT NULL,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_goals_user
        FOREIGN KEY (user_id)
        REFERENCES users(id)
        ON DELETE CASCADE,

    -- ห้าม user ตั้ง goal ซ้ำในเดือนเดียว
    CONSTRAINT uniq_user_month_goal
        UNIQUE (user_id, month)
        
        
    
     CREATE TABLE user_profiles (
    id UUID PRIMARY KEY,
    user_id UUID UNIQUE NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    display_name VARCHAR(150),
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(30),
    profile_image_url TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
