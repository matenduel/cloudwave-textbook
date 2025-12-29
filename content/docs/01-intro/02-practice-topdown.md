---
title: "Docker Practice (Top-Down)"
weight: 20
draft: false
---

# Docker (Top-Down) practice



## 1. 실습용 IDE 설치하기

### 상황 설명

- 개발 편의성을 위해서 `dev container`를 쓰는 경우가 많습니다.
  - 개발 환경을 코드로 고정해서(재현성) 누구나 같은 조건에서 빠르게 개발·테스트·리뷰하게 만듭니다.



### How to

**Command line**

```sh
docker volume create config


# Window
docker run -d --name ide --network host -e PASSWORD=wave -e DEFAULT_WORKSPACE=/code -e PUID=0 -e PGID=0 -e TZ="Asia/Seoul" -v ${home}\src:/code -v /var/run/docker.sock:/var/run/docker.sock -v config:/config linuxserver/code-server:4.107.0
```



- `localhost:8443`에 접속합니다.
  - Password = `wave`

![image-20251229204547145](/images/docker-practice-setting/image-20251229204547145.png)

- 초기 화면은 다음과 같습니다. 

![image-20251229204645897](/images/docker-practice-setting/image-20251229204645897.png)



- 필요한 `Extensions`를 설치합니다. 
  - Docker, YAML

![image-20251229204724640](/images/docker-practice-setting/image-20251229204724640.png)



## 2. Metabase를 이용하여 데이터 시각화하기

### 상황 설명

- 데이터 시각화를 위한 `metabase` 도입을 결정하기 위해 POC를 진행하는 상황입니다.



### How to

**Command line**

```sh
# Docker network 생성
docker network create -d bridge private

# Docker volume 생성
docker volume create db_data

# PostgreSQL DB 생성
docker run --rm -d --name db --network private -v db_data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 postgres:16.1-bullseye

# meta base
docker run -d --name metabase -p 3000:3000 --network private metabase/metabase:latest
```

- `localhost:3000`에 접속합니다.
- 초기 설정을 진행합니다. 
  - 이름, 이메일은 임의로 작성하면 됩니다. 
- 다음 정보를 이용하여 DB를 연결합니다. 
  - `jdbc:postgresql://db:5432/postgres`
- DB 데이터를 GUI로 변경하고 싶다면 다음 명령어를 통해 `PgAdmin` 서비스를 실행합니다.
  - `localhost:8080`으로 접속합니다.
  - 로그인 정보 (user@sample.com / SuperSecret)

```sh
# PgAdmin Application 생성
docker run --rm -d -p 8080:80 --name pgadmin -e PGADMIN_DEFAULT_EMAIL=user@sample.com -e PGADMIN_DEFAULT_PASSWORD=SuperSecret --network private dpage/pgadmin4:7.4
```



### **DB Data 입력**

```sql
-- =========================================================
-- Metabase 연습용 샘플 데이터: 간단 이커머스(주문/고객/상품)
-- PostgreSQL 전용 (generate_series, setseed 사용)
-- =========================================================

BEGIN;

-- 재실행 편의: 기존 객체 제거
DROP VIEW IF EXISTS v_order_summary;
DROP TABLE IF EXISTS payments;
DROP TABLE IF EXISTS order_items;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS customers;

-- -------------------------
-- 1) 테이블 생성 (DDL)
-- -------------------------
CREATE TABLE customers (
  customer_id      BIGSERIAL PRIMARY KEY,
  full_name        TEXT NOT NULL,
  email            TEXT UNIQUE NOT NULL,
  signup_date      DATE NOT NULL,
  region           TEXT NOT NULL,
  city             TEXT NOT NULL,
  age_band         TEXT NOT NULL,   -- '18-24' 등
  is_b2b           BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE TABLE products (
  product_id       BIGSERIAL PRIMARY KEY,
  product_name     TEXT NOT NULL,
  category         TEXT NOT NULL,
  price            NUMERIC(10,2) NOT NULL CHECK (price > 0)
);

CREATE TABLE orders (
  order_id         BIGSERIAL PRIMARY KEY,
  customer_id      BIGINT NOT NULL REFERENCES customers(customer_id),
  order_ts         TIMESTAMPTZ NOT NULL,
  channel          TEXT NOT NULL,     -- web/mobile/partner
  status           TEXT NOT NULL,     -- paid/shipped/delivered/cancelled
  discount_rate    NUMERIC(5,2) NOT NULL DEFAULT 0 CHECK (discount_rate >= 0 AND discount_rate <= 0.80),
  shipping_fee     NUMERIC(10,2) NOT NULL DEFAULT 0 CHECK (shipping_fee >= 0)
);

CREATE TABLE order_items (
  order_item_id    BIGSERIAL PRIMARY KEY,
  order_id         BIGINT NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
  product_id       BIGINT NOT NULL REFERENCES products(product_id),
  quantity         INT NOT NULL CHECK (quantity > 0),
  unit_price       NUMERIC(10,2) NOT NULL CHECK (unit_price > 0)
);

CREATE TABLE payments (
  payment_id       BIGSERIAL PRIMARY KEY,
  order_id         BIGINT NOT NULL UNIQUE REFERENCES orders(order_id) ON DELETE CASCADE,
  paid_ts          TIMESTAMPTZ NOT NULL,
  method           TEXT NOT NULL,        -- card/kakao/bank/virtual
  amount_paid      NUMERIC(12,2) NOT NULL CHECK (amount_paid >= 0),
  is_refunded      BOOLEAN NOT NULL DEFAULT FALSE,
  refunded_ts      TIMESTAMPTZ NULL
);

-- 인덱스(조회 연습/성능 체감용)
CREATE INDEX idx_orders_order_ts ON orders(order_ts);
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_items_order ON order_items(order_id);
CREATE INDEX idx_products_category ON products(category);

-- -------------------------
-- 2) 샘플 데이터 입력
-- -------------------------

-- 난수 재현성(실행할 때마다 비슷한 분포)
SELECT setseed(0.42);

-- (A) 상품 20개
INSERT INTO products (product_name, category, price) VALUES
('Cloud Notebook',     'stationery',  9.90),
('Gel Pen Set',        'stationery',  5.50),
('Desk Lamp',          'home',       24.90),
('Ergonomic Chair Pad','home',       19.90),
('USB-C Hub',          'electronics', 29.90),
('Mechanical Keyboard','electronics', 79.00),
('Noise-Cancel Earbuds','electronics',59.00),
('Monitor Stand',      'home',       34.00),
('Water Bottle',       'lifestyle',  12.00),
('Yoga Mat',           'lifestyle',  22.00),
('Protein Bar Box',    'food',       18.50),
('Coffee Beans 1kg',   'food',       28.00),
('T-shirt',            'apparel',    15.00),
('Hoodie',             'apparel',    39.00),
('Socks Pack',         'apparel',     8.00),
('Book: Dev Basics',   'books',      21.00),
('Book: Data Viz',     'books',      27.00),
('Planner 2026',       'stationery', 14.00),
('Smart Plug',         'electronics',17.50),
('Aroma Diffuser',     'home',       26.00);

-- (B) 고객 60명 (지역/도시/연령대 포함)
WITH regions AS (
  SELECT unnest(ARRAY[
    'Seoul','Gyeonggi','Incheon','Busan','Daegu','Daejeon','Gwangju'
  ]) AS region
),
cities AS (
  SELECT unnest(ARRAY[
    'Gangnam','Mapo','Songpa','Suwon','Seongnam','Bupyeong','Haeundae','Suseong','Yuseong'
  ]) AS city
),
age_bands AS (
  SELECT unnest(ARRAY['18-24','25-34','35-44','45-54','55+']) AS age_band
)
INSERT INTO customers (full_name, email, signup_date, region, city, age_band, is_b2b)
SELECT
  'Customer ' || gs AS full_name,
  'customer' || gs || '@example.com' AS email,
  (CURRENT_DATE - ((random() * 180)::int))::date AS signup_date,
  (SELECT region FROM regions ORDER BY random() LIMIT 1) AS region,
  (SELECT city FROM cities ORDER BY random() LIMIT 1) AS city,
  (SELECT age_band FROM age_bands ORDER BY random() LIMIT 1) AS age_band,
  (random() < 0.15) AS is_b2b
FROM generate_series(1, 60) gs;

-- (C) 주문 120건: 최근 120일 범위에 분포
WITH channels AS (
  SELECT unnest(ARRAY['web','mobile','partner']) AS channel
),
statuses AS (
  -- paid 45%, shipped 25%, delivered 20%, cancelled 10% 정도
  SELECT
    CASE
      WHEN r < 0.10 THEN 'cancelled'
      WHEN r < 0.55 THEN 'paid'
      WHEN r < 0.80 THEN 'shipped'
      ELSE 'delivered'
    END AS status
  FROM (SELECT random() AS r) t
)
INSERT INTO orders (customer_id, order_ts, channel, status, discount_rate, shipping_fee)
SELECT
  (1 + floor(random() * 60))::bigint AS customer_id,
  now() - ((random() * 120)::int || ' days')::interval - ((random() * 86400)::int || ' seconds')::interval AS order_ts,
  (SELECT channel FROM channels ORDER BY random() LIMIT 1) AS channel,
  (SELECT status  FROM statuses LIMIT 1) AS status,
  -- 할인율: 0, 5%, 10%, 15%, 20% 중 하나
  (ARRAY[0.00,0.05,0.10,0.15,0.20])[1 + floor(random() * 5)]::numeric AS discount_rate,
  -- 배송비: 파트너는 무료가 더 많고, 나머지는 0/2.5/3.5 중 랜덤
  CASE
    WHEN (random() < 0.35) THEN 0
    ELSE (ARRAY[2.50,3.50])[1 + floor(random() * 2)]::numeric
  END AS shipping_fee
FROM generate_series(1, 120);

-- (D) 주문 아이템 240건 (주문당 평균 2개꼴)
-- order_id는 1..120이라고 가정(비어있는 DB에서 처음 넣는다는 전제)
INSERT INTO order_items (order_id, product_id, quantity, unit_price)
SELECT
  (1 + (gs % 120))::bigint AS order_id,
  (1 + floor(random() * 20))::bigint AS product_id,
  (1 + floor(random() * 4))::int AS quantity,
  p.price AS unit_price
FROM generate_series(1, 240) gs
JOIN products p ON p.product_id = (1 + floor(random() * 20))::bigint;

-- (E) 결제: cancelled는 결제 없게, 일부는 환불 처리
-- amount_paid는 아이템 합계 * (1-discount) + shipping
WITH order_totals AS (
  SELECT
    o.order_id,
    o.order_ts,
    o.status,
    o.discount_rate,
    o.shipping_fee,
    SUM(oi.quantity * oi.unit_price) AS items_gross
  FROM orders o
  JOIN order_items oi ON oi.order_id = o.order_id
  GROUP BY o.order_id, o.order_ts, o.status, o.discount_rate, o.shipping_fee
),
methods AS (
  SELECT unnest(ARRAY['card','kakao','bank','virtual']) AS method
)
INSERT INTO payments (order_id, paid_ts, method, amount_paid, is_refunded, refunded_ts)
SELECT
  ot.order_id,
  ot.order_ts + ((random() * 7200)::int || ' seconds')::interval AS paid_ts,
  (SELECT method FROM methods ORDER BY random() LIMIT 1) AS method,
  ROUND((ot.items_gross * (1 - ot.discount_rate) + ot.shipping_fee)::numeric, 2) AS amount_paid,
  -- paid/shipped/delivered 중 8% 환불
  (ot.status <> 'cancelled' AND random() < 0.08) AS is_refunded,
  CASE
    WHEN (ot.status <> 'cancelled' AND random() < 0.08)
      THEN ot.order_ts + ((random() * 7)::int || ' days')::interval
    ELSE NULL
  END AS refunded_ts
FROM order_totals ot
WHERE ot.status <> 'cancelled';

-- -------------------------
-- 3) Metabase 친화 뷰(추천)
-- -------------------------
CREATE VIEW v_order_summary AS
SELECT
  o.order_id,
  o.order_ts,
  o.channel,
  o.status,
  c.region,
  c.city,
  c.age_band,
  c.is_b2b,
  COUNT(DISTINCT oi.product_id) AS distinct_products,
  SUM(oi.quantity) AS total_qty,
  ROUND(SUM(oi.quantity * oi.unit_price)::numeric, 2) AS items_gross,
  o.discount_rate,
  ROUND((SUM(oi.quantity * oi.unit_price) * (1 - o.discount_rate))::numeric, 2) AS items_net,
  o.shipping_fee,
  ROUND((SUM(oi.quantity * oi.unit_price) * (1 - o.discount_rate) + o.shipping_fee)::numeric, 2) AS order_total,
  p.amount_paid,
  p.method AS payment_method,
  p.is_refunded
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
JOIN order_items oi ON oi.order_id = o.order_id
LEFT JOIN payments p ON p.order_id = o.order_id
GROUP BY
  o.order_id, o.order_ts, o.channel, o.status,
  c.region, c.city, c.age_band, c.is_b2b,
  o.discount_rate, o.shipping_fee,
  p.amount_paid, p.method, p.is_refunded;

COMMIT;

-- 데이터 개수 확인(옵션)
-- SELECT 'customers' AS t, COUNT(*) FROM customers
-- UNION ALL SELECT 'products', COUNT(*) FROM products
-- UNION ALL SELECT 'orders', COUNT(*) FROM orders
-- UNION ALL SELECT 'order_items', COUNT(*) FROM order_items
-- UNION ALL SELECT 'payments', COUNT(*) FROM payments;

```









## 3. “S3 스토리지 서버”를 내 PC에 띄우기 (MinIO)



### How to

**Bash**

```sh
docker volume create minio-data

# 최신 버젼 (Dashboard X)
docker run -d --name minio -p 9000:9000 -p 9001:9001 -e MINIO_ROOT_USER=minio -e MINIO_ROOT_PASSWORD=minio1234 -v minio-data:/data minio/minio server /data --console-address ":9001"


# 구 버젼 (Dashboard O)
docker run -d --name minio -p 9000:9000 -p 9001:9001 -e MINIO_ROOT_USER=minio -e MINIO_ROOT_PASSWORD=minio1234 minio/minio:RELEASE.2025-04-22T22-12-26Z server /data --console-address ":9001"
```

### 확인

- API: `http://localhost:9000`
- 콘솔: `http://localhost:9001`
- 계정: `minio / minio1234`
- 대시보드: `http://localhost:9001/tools/metrics`



![image-20251229210245329](/images/docker-practice-setting/image-20251229210245329.png)