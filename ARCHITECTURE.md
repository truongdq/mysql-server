# MySQL Server - Kiến Trúc & Hướng Dẫn Build

## Thông Tin Chung

- **Project**: MySQL Server
- **Phiên bản**: 9.5.0 INNOVATION
- **Build System**: CMake (≥ 3.17.5)
- **Ngôn ngữ**: C/C++
- **License**: GNU General Public License v2.0

## Kiến Trúc Project

### Cấu Trúc Thư Mục Chính

| Thư mục | Mô tả |
|---------|-------|
| `sql/` | Core Server - SQL parser, optimizer, executor |
| `storage/` | Storage Engines - InnoDB, MyISAM, NDB... |
| `client/` | Client Tools - mysql, mysqladmin, mysqldump... |
| `libmysql/` | Client Library - C API |
| `plugin/` | Plugins - Authentication, replication, audit |
| `components/` | Server Components - Keyring, validate_password |
| `router/` | MySQL Router - Proxy/load balancer |
| `include/` | Public Header Files |
| `mysys/` | System Library - OS abstraction layer |
| `strings/` | String Library - Character set handling |
| `vio/` | Virtual I/O - Network abstraction |
| `sql-common/` | Shared SQL Code - Client/Server |
| `extra/` | Third-party Libraries - Boost, protobuf, zlib... |
| `mysql-test/` | Test Suite - Regression tests |
| `unittest/` | Unit Tests - Google Test |
| `scripts/` | SQL Scripts - System schemas |
| `cmake/` | Build Configuration - CMake modules |
| `packaging/` | Packaging - RPM, DEB, Docker |

### Storage Engines

```
storage/
├── innobase/     # InnoDB - Default transactional engine
├── myisam/       # MyISAM - Legacy non-transactional
├── ndb/          # NDB Cluster - Distributed storage
├── temptable/    # TempTable - In-memory temporary tables
├── perfschema/   # Performance Schema
├── heap/         # MEMORY engine
├── federated/    # Federated tables
├── archive/      # Archive engine
├── blackhole/    # Blackhole engine
└── csv/          # CSV engine
```

## Yêu Cầu Hệ Thống

- **CMake**: ≥ 3.17.5
- **Windows**: Visual Studio 2019/2022 (64-bit), Windows 10+
- **Linux**: GCC hoặc Clang
- **macOS**: Xcode, CMake ≥ 3.19
- **OpenSSL**: Header và library

## Hướng Dẫn Build

### Windows

```powershell
# Tạo thư mục build
mkdir build
cd build

# Configure với Visual Studio 2022
cmake .. -G "Visual Studio 17 2022" -A x64

# Build Debug
cmake --build . --config Debug

# Build Release
cmake --build . --config RelWithDebInfo

# Build song song (nhanh hơn)
cmake --build . --config Debug -- -m

# Build im lặng + song song
cmake --build . --config Debug -- -m -v:q
```

### Linux / macOS

```bash
# Tạo thư mục build
mkdir build && cd build

# Configure
cmake ..

# Build
make -j$(nproc)

# Build với debug
cmake -DWITH_DEBUG=1 ..
make -j$(nproc)
```

### Build với Clang trên Windows (Experimental)

```powershell
cmake -G Ninja -DFORCE_UNSUPPORTED_COMPILER=1 `
  -DCMAKE_C_COMPILER="C:/Program Files/LLVM/bin/clang-cl.exe" `
  -DCMAKE_CXX_COMPILER="C:/Program Files/LLVM/bin/clang-cl.exe" `
  -DCMAKE_LINKER="C:/Program Files/LLVM/bin/lld-link.exe" `
  ..

ninja
```

## CMake Options

| Option | Mô tả | Default |
|--------|-------|---------|
| `WITH_DEBUG` | Build debug mode | OFF |
| `WITH_SSL` | Đường dẫn OpenSSL | system |
| `MAX_INDEXES` | Số index tối đa/table | 64 |
| `WITH_DEFAULT_COMPILER_OPTIONS` | Sử dụng compiler options mặc định | ON |
| `WITH_ASAN` | Address Sanitizer | OFF |
| `FORCE_UNSUPPORTED_COMPILER` | Bỏ qua kiểm tra compiler | OFF |

## Chạy MySQL Server

```bash
# Khởi tạo data directory
bin/mysqld --initialize --datadir=/path/to/data

# Chạy server
bin/mysqld --datadir=/path/to/data

# Kết nối client
bin/mysql -u root -p
```

## Chạy Tests

```bash
cd mysql-test

# Chạy test suite
perl mysql-test-run.pl

# Chạy với parallel
perl mysql-test-run.pl --parallel=4 --build-thread=500
```

## Lưu Ý

- Codebase: **30,000+ files**
- Build time: **30-60 phút** (lần đầu)
- Disk space: **20-50GB** (full build)

## Tài Liệu Tham Khảo

- [MySQL Documentation](https://dev.mysql.com/doc/refman/en/)
- [Source Installation](https://dev.mysql.com/doc/refman/en/source-installation.html)
- [MySQL Downloads](https://www.mysql.com/downloads)

