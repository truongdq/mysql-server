# Giai Đoạn 8: MyISAM, Memory, TempTable (Tháng 11)

## Mục Tiêu
- Hiểu các storage engines khác ngoài InnoDB
- So sánh đặc điểm và use cases
- Biết khi nào nên dùng engine nào

---

## Tổng Quan

| Feature | InnoDB | MyISAM | Memory | TempTable |
|---------|--------|--------|--------|-----------|
| Transactions | Yes | No | No | No |
| Locking | Row | Table | Table | Table |
| MVCC | Yes | No | No | No |
| Foreign Keys | Yes | No | No | No |
| Crash Recovery | Yes | No | No | N/A |
| Full-text | Yes | Yes | No | No |
| Persistence | Yes | Yes | No | No |
| Use case | OLTP | Read-heavy | Cache | Temp tables |

---

## MyISAM Engine

### Thư mục: `storage/myisam/`

### Files Quan Trọng

| File | Chức năng |
|------|-----------|
| `mi_open.c` | Open table |
| `mi_close.c` | Close table |
| `mi_write.c` | Insert row |
| `mi_update.c` | Update row |
| `mi_delete.c` | Delete row |
| `mi_search.c` | Index search |
| `mi_key.c` | Key handling |
| `mi_cache.c` | Key cache |

### Cấu Trúc File

Mỗi MyISAM table có 3 files:
```
table_name.frm  → Table definition (deprecated in 8.0)
table_name.MYD  → Data file
table_name.MYI  → Index file
```

### Key Cache

MyISAM cache index blocks (không cache data):
```cpp
struct st_key_cache {
    byte *block_mem;      // Memory for blocks
    BLOCK_LINK *block_root; // Block hash
    // ...
};
```

**Configuration:**
```sql
SET GLOBAL key_buffer_size = 256M;
```

### Bài Tập MyISAM

#### Bài 1: Create và Trace
```sql
CREATE TABLE test_myisam (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    INDEX idx_name (name)
) ENGINE=MyISAM;

INSERT INTO test_myisam VALUES (1, 'Alice');
SELECT * FROM test_myisam WHERE id = 1;
```

**Breakpoints:**
1. `ha_myisam::write_row()` - storage/myisam/ha_myisam.cc
2. `mi_write()` - storage/myisam/mi_write.c
3. `mi_search()` - storage/myisam/mi_search.c

#### Bài 2: Table Locking
```sql
-- Session 1
LOCK TABLES test_myisam WRITE;
-- Do something
UNLOCK TABLES;

-- Session 2 (blocked)
SELECT * FROM test_myisam;
```

**Observe:**
- Lock wait behavior
- Không có row-level locking

#### Bài 3: Full-text Search
```sql
CREATE TABLE articles (
    id INT PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    FULLTEXT(title, content)
) ENGINE=MyISAM;

INSERT INTO articles VALUES (1, 'MySQL Tutorial', 'MySQL is a database...');
SELECT * FROM articles WHERE MATCH(title, content) AGAINST('MySQL');
```

---

## Memory/Heap Engine

### Thư mục: `storage/heap/`

### Files Quan Trọng

| File | Chức năng |
|------|-----------|
| `hp_open.c` | Open table |
| `hp_close.c` | Close table |
| `hp_write.c` | Insert row |
| `hp_update.c` | Update row |
| `hp_delete.c` | Delete row |
| `hp_hash.c` | Hash index |
| `hp_rfirst.c` | Index scan |

### Đặc Điểm

- **In-memory only**: Data mất khi restart
- **Fixed row length**: Không hỗ trợ BLOB, TEXT
- **Hash index** (default) hoặc **B-Tree**
- **Table-level locking**

### Index Types

```sql
-- Hash index (default, O(1) lookup)
CREATE TABLE mem_hash (
    id INT,
    name VARCHAR(100),
    INDEX USING HASH (id)
) ENGINE=MEMORY;

-- B-Tree index (range queries)
CREATE TABLE mem_btree (
    id INT,
    name VARCHAR(100),
    INDEX USING BTREE (id)
) ENGINE=MEMORY;
```

### Bài Tập Memory Engine

#### Bài 1: Hash vs B-Tree
```sql
CREATE TABLE mem_test (
    id INT,
    value INT,
    INDEX USING HASH (id),
    INDEX USING BTREE (value)
) ENGINE=MEMORY;

-- Hash: exact match only
EXPLAIN SELECT * FROM mem_test WHERE id = 1;

-- B-Tree: range queries
EXPLAIN SELECT * FROM mem_test WHERE value > 100;
```

#### Bài 2: Memory Limits
```sql
SET max_heap_table_size = 16M;

-- Insert until limit reached
INSERT INTO mem_test SELECT * FROM large_table;
-- Error: Table is full
```

#### Bài 3: Trace Hash Lookup
**Breakpoints:**
1. `ha_heap::index_read()` - storage/heap/ha_heap.cc
2. `hp_search()` - storage/heap/hp_hash.c

---

## TempTable Engine

### Thư mục: `storage/temptable/`

### Files Quan Trọng

| File | Chức năng |
|------|-----------|
| `handler.cc` | Handler implementation |
| `table.cc` | Table operations |
| `row.cc` | Row handling |
| `index.cc` | Index structures |
| `allocator.cc` | Memory allocation |

### Đặc Điểm

- Optimized cho internal temporary tables
- Variable-length rows (unlike MEMORY)
- Hash và B-Tree indexes
- Memory-efficient allocator
- Mặc định từ MySQL 8.0

### Khi Nào Dùng TempTable

MySQL tự động tạo temp tables khi:
- GROUP BY với columns không có index
- ORDER BY và GROUP BY khác columns
- DISTINCT với ORDER BY
- UNION queries
- Derived tables, subqueries

```sql
-- Check temp table usage
SHOW STATUS LIKE 'Created_tmp%';
```

### Configuration
```sql
-- TempTable memory limit
SET GLOBAL temptable_max_ram = 1G;

-- Fallback to disk
SET GLOBAL temptable_max_mmap = 1G;
```

### Bài Tập TempTable

#### Bài 1: Observe Temp Table Creation
```sql
EXPLAIN SELECT department, COUNT(*) 
FROM employees 
GROUP BY department 
ORDER BY COUNT(*) DESC;
```

**Check Extra column cho "Using temporary"**

#### Bài 2: Trace Internal Temp Table
**Breakpoints:**
1. `create_tmp_table()` - sql/sql_tmp_table.cc
2. `ha_temptable::create()` - storage/temptable/handler.cc

---

## So Sánh Performance

### Test Setup
```sql
CREATE TABLE test_innodb (id INT PRIMARY KEY, val INT) ENGINE=InnoDB;
CREATE TABLE test_myisam (id INT PRIMARY KEY, val INT) ENGINE=MyISAM;
CREATE TABLE test_memory (id INT PRIMARY KEY, val INT) ENGINE=MEMORY;

-- Insert 100K rows each
```

### Benchmark Tests

| Test | InnoDB | MyISAM | Memory |
|------|--------|--------|--------|
| Single INSERT | Medium | Fast | Fastest |
| Bulk INSERT | Fast | Fastest | Fast |
| PK Lookup | Fast | Fast | Fastest |
| Range Scan | Fast | Medium | Slow (hash) |
| Concurrent Read | Excellent | Good | Good |
| Concurrent Write | Good | Poor | Poor |

### When to Use Each

| Engine | Use Case |
|--------|----------|
| InnoDB | Default choice, OLTP, transactions |
| MyISAM | Read-heavy, full-text (legacy) |
| Memory | Session cache, lookup tables |
| TempTable | Internal temp tables (automatic) |

---

## Bài Tập Tổng Hợp

### Bài 1: Engine Comparison
1. Tạo same table với 3 engines
2. Insert 10K rows vào mỗi table
3. Benchmark SELECT, UPDATE với concurrent connections
4. Viết report so sánh

### Bài 2: Trace Same Query on Different Engines
Query: `SELECT * FROM table WHERE id BETWEEN 100 AND 200`

Trace trên InnoDB, MyISAM, Memory và note:
- Handler methods called
- Lock behavior
- Buffer/cache usage

---

## Checklist Cuối Tháng 11

- [ ] Hiểu MyISAM architecture (MYD, MYI files)
- [ ] Hiểu key cache trong MyISAM
- [ ] Hiểu Memory engine (hash vs btree)
- [ ] Hiểu TempTable purpose
- [ ] So sánh được performance các engines
- [ ] Biết khi nào dùng engine nào

