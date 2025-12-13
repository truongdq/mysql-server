# Giai Đoạn 1: Nền Tảng (Tháng 1-2)

## Mục Tiêu
- Nắm vững C/C++ ở mức có thể đọc hiểu source code MySQL
- Setup môi trường và build MySQL Server thành công
- Chạy MySQL local và thực hiện các thao tác cơ bản

---

## Tháng 1: C/C++ Foundation

### Kiến Thức Bắt Buộc

| Chủ đề | Mức độ | Tài liệu |
|--------|--------|----------|
| Pointers & References | Thành thạo | C++ Primer Ch. 2, 12 |
| Memory Management | malloc/free, new/delete, RAII | C++ Primer Ch. 12 |
| STL Containers | vector, map, set, unordered_map | C++ Primer Ch. 9, 11 |
| OOP in C++ | Class, inheritance, virtual | C++ Primer Ch. 13-15 |
| Smart Pointers | unique_ptr, shared_ptr | C++ Primer Ch. 12 |
| Templates | Basic templates | C++ Primer Ch. 16 |

### Bài Tập Thực Hành

#### Tuần 1-2: Pointers & Memory
```cpp
// Bài 1: Viết hàm swap 2 số dùng pointer
void swap(int* a, int* b);

// Bài 2: Tạo dynamic array, resize, và giải phóng
int* createArray(int size);
int* resizeArray(int* arr, int oldSize, int newSize);
void freeArray(int* arr);

// Bài 3: Implement linked list đơn giản
struct Node { int data; Node* next; };
```

#### Tuần 3: STL Containers
```cpp
// Bài 4: Sử dụng vector, map để đếm từ trong file
std::map<std::string, int> countWords(const std::string& filename);

// Bài 5: Implement LRU Cache với unordered_map + list
class LRUCache {
    int get(int key);
    void put(int key, int value);
};
```

#### Tuần 4: OOP & Smart Pointers
```cpp
// Bài 6: Implement class hierarchy
class Shape { virtual double area() = 0; };
class Circle : public Shape { ... };
class Rectangle : public Shape { ... };

// Bài 7: Convert raw pointers sang smart pointers
std::unique_ptr<Shape> createShape(const std::string& type);
```

### Checklist Tuần 4
- [ ] Hiểu pointer arithmetic
- [ ] Có thể debug memory leak
- [ ] Sử dụng thành thạo vector, map
- [ ] Hiểu virtual function và polymorphism
- [ ] Biết khi nào dùng unique_ptr vs shared_ptr

---

## Tháng 2: Setup và Build MySQL

### Yêu Cầu Phần Mềm

| Phần mềm | Version | Download |
|----------|---------|----------|
| Visual Studio | 2022 | https://visualstudio.microsoft.com/ |
| CMake | ≥ 3.17.5 | https://cmake.org/download/ |
| Git | Latest | https://git-scm.com/ |
| OpenSSL | Latest | Có sẵn trong VS |

### Bước 1: Clone Source Code
```powershell
git clone https://github.com/mysql/mysql-server.git
cd mysql-server
git checkout mysql-9.0.0  # hoặc version mới nhất
```

### Bước 2: Tạo Build Directory
```powershell
mkdir build
cd build
```

### Bước 3: Configure với CMake
```powershell
cmake .. -G "Visual Studio 17 2022" -A x64
```

### Bước 4: Build (Debug Mode)
```powershell
cmake --build . --config Debug -- -m -v:q
```
> Lưu ý: Build lần đầu mất 30-60 phút

### Bước 5: Khởi Tạo Data Directory
```powershell
.\runtime_output_directory\Debug\mysqld.exe --initialize --datadir=D:\mysql-data
```
> Ghi lại temporary password được in ra

### Bước 6: Chạy Server
```powershell
.\runtime_output_directory\Debug\mysqld.exe --datadir=D:\mysql-data --console
```

### Bước 7: Kết Nối Client
```powershell
.\runtime_output_directory\Debug\mysql.exe -u root -p
```

### Thư Mục Cần Làm Quen

| Thư mục | Mô tả | Độ ưu tiên |
|---------|-------|------------|
| `cmake/` | Build configuration | Cao |
| `include/` | Public headers | Cao |
| `mysys/` | OS abstraction layer | Trung bình |
| `sql/` | Core SQL engine | Xem qua |
| `storage/` | Storage engines | Xem qua |

### Bài Tập

1. **Build thành công**: Screenshot kết quả build
2. **Chạy server**: Kết nối và chạy `SELECT VERSION();`
3. **Khám phá CMake**: Đọc `CMakeLists.txt` gốc, liệt kê 10 options quan trọng
4. **Debug đầu tiên**: Đặt breakpoint trong `main()` của mysqld, trace 5 bước đầu

### Checklist Cuối Tháng 2
- [ ] Build MySQL thành công (Debug mode)
- [ ] Chạy được MySQL server local
- [ ] Kết nối được bằng mysql client
- [ ] Hiểu cấu trúc thư mục cơ bản
- [ ] Có thể đặt breakpoint và debug cơ bản

---

## Troubleshooting

### Lỗi thường gặp khi Build

| Lỗi | Nguyên nhân | Giải pháp |
|-----|-------------|-----------|
| OpenSSL not found | Thiếu OpenSSL | Cài từ VS Installer |
| CMake version | CMake cũ | Update CMake |
| Out of memory | RAM không đủ | Tắt browser, build -j2 |
| Disk space | Thiếu disk | Cần 20-50GB free |

### Lỗi khi chạy Server

| Lỗi | Nguyên nhân | Giải pháp |
|-----|-------------|-----------|
| Can't create directory | Permission | Chạy as Admin hoặc đổi path |
| Port in use | Port 3306 đã dùng | Thêm --port=3307 |
| Data directory exists | Đã initialize | Xóa data dir hoặc skip |

