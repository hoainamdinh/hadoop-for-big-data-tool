# 📚 Hướng Dẫn Cài Đặt Big Data Stack

> **Hadoop 2.7.4 • Hive 2.3.2 • Hue 4.6.0 • Spark 3.5.1 • Python 3.10**

Tài liệu hướng dẫn chi tiết cách cài đặt và quản lý môi trường Big Data hoàn chỉnh sử dụng Docker Compose.

---

## 📋 Mục Lục

- [🏗️ Kiến Trúc Hệ Thống](#️-kiến-trúc-hệ-thống)
- [⚙️ Yêu Cầu Hệ Thống](#️-yêu-cầu-hệ-thống)
- [🚀 Khởi Động Nhanh](#-khởi-động-nhanh)
- [🌐 Địa Chỉ Dịch Vụ & Cổng](#-địa-chỉ-dịch-vụ--cổng)
- [🛠️ Cấu Hình Phím Tắt Bash](#️-cấu-hình-phím-tắt-bash)
- [📖 Hướng Dẫn Sử Dụng](#-hướng-dẫn-sử-dụng)
- [⚠️ Lưu Ý Quan Trọng](#️-lưu-ý-quan-trọng)

---

## 🏗️ Kiến Trúc Hệ Thống

```
┌─────────────────────────────────────────────────────────────┐
│                 bigdata-net (Docker Network)                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌───────────────────────────────────────────────────┐     │
│  │               Hadoop Hive Hue                     │     │
│  │   📦 Hadoop NameNode (Lưu trữ HDFS)               │     │
│  │   📦 Hadoop DataNode                              │     │
│  │   📦 YARN ResourceManager                         │     │
│  │   📦 Hive Metastore + PostgreSQL                  │     │
│  │   📦 HiveServer2 (JDBC:10000)                     │     │
│  │   📦 Hue Web UI (Trình soạn SQL)                  │     │
│  └───────────────────────────────────────────────────┘     │
│                          ↕                                  │
│  ┌───────────────────────────────────────────────────┐     │
│  │        Spark Cluster                              │     │
│  │   ⚡ Spark Master (Cổng 7077)                     │     │
│  │   ⚡ Spark Worker (Python 3.10)                   │     │
│  │   🔗 Kết nối với HDFS & Hive                      │     │
│  └───────────────────────────────────────────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## ⚙️ Yêu Cầu Hệ Thống

| Yêu cầu | Chi tiết |
|---------|----------|
| **Hệ điều hành** | Linux (Ubuntu 20.04+) / WSL2 trên Windows |
| **Docker Engine** | Phiên bản 20.10+ |
| **Docker Compose** | Phiên bản 1.29+ hoặc V2 |
| **RAM** | Tối thiểu 8GB (khuyến nghị 16GB) |
| **Ổ cứng** | Ít nhất 20GB trống |

---

## 🚀 Khởi Động Nhanh

### Bước 1: Clone Project

Tải mã nguồn về máy:

```bash
cd ~
git clone https://github.com/hoainamdinh/hadoop-for-big-data-tool.git
cd hadoop-for-big-data-tool
```

### Bước 2: Tạo Docker Network

Tất cả dịch vụ giao tiếp qua một mạng chung:

```bash
docker network create bigdata-net
```

### Bước 3: Khởi Động Dịch Vụ

**Cách A - Khởi động thủ công:**
```bash
# Khởi động Hadoop + Hive + Hue
cd ~/hadoop-for-big-data-tool/docker-hadoop-hive-hue && docker-compose up -d

# Khởi động Spark
cd ~/hadoop-for-big-data-tool/docker-spark && docker-compose up -d
```

**Cách B - Dùng phím tắt Bash** *(khuyến nghị)*:
```bash
start-bigdata
```

### Bước 4: Kiểm Tra Dịch Vụ

Chờ 2-3 phút để khởi tạo xong, sau đó kiểm tra:

```bash
docker ps
show-links
```

---

## 🌐 Địa Chỉ Dịch Vụ & Cổng

| 🔧 Dịch vụ | 🌍 Địa chỉ | 📌 Cổng | 📝 Mô tả |
|------------|------------|---------|----------|
| **Hadoop NameNode** | http://localhost:50070 | `50070` | Giao diện HDFS - Duyệt file, kiểm tra cluster |
| **Hadoop DataNode** | http://localhost:50075 | `50075` | Giám sát DataNode |
| **🎨 Hue Interface** | http://localhost:8888 | `8888` | **Trình soạn SQL & Duyệt file chính** |
| **⚡ Spark Master** | http://localhost:8080 | `8080` | Dashboard Spark cluster |
| **Spark RPC** | spark://localhost:7077 | `7077` | Điểm submit Spark job |
| **Hive JDBC** | jdbc:hive2://localhost:10000 | `10000` | Kết nối trực tiếp Hive |
| **Hive Metastore** | thrift://localhost:9083 | `9083` | Dịch vụ metastore nội bộ |
| **PostgreSQL** | localhost:5432 | `5432` | Lưu trữ metadata |

### 🔐 Thông Tin Đăng Nhập Mặc Định

| Dịch vụ | Tên đăng nhập | Mật khẩu |
|---------|---------------|----------|
| **Hue** | `admin` | `admin` *(tạo mới lần đầu đăng nhập)* |
| **PostgreSQL** | `hive` | `hive` |

---

## 🛠️ Cấu Hình Phím Tắt Bash

Thêm các hàm sau vào file `~/.bashrc` để quản lý tiện lợi:

```bash
# ==========================================
# 🎯 PHÍM TẮT BIG DATA STACK
# ==========================================

# 🚀 Khởi động toàn bộ Big Data stack
start-bigdata() {
    echo "🌐 Đang tạo network..."
    docker network create bigdata-net 2>/dev/null || echo "✅ Network đã tồn tại"
    
    echo "🐘 Đang khởi động Hadoop + Hive + Hue..."
    (cd "~/hadoop-for-big-data-tool/docker-hadoop-hue-hive" && docker-compose up -d)
    
    echo "⚡ Đang khởi động Spark cluster..."
    (cd "~/hadoop-for-big-data-tool/docker-spark" && docker-compose up -d)
    
    echo "✅ Đã khởi động xong! Chờ 2-3 phút để các dịch vụ sẵn sàng."
    show-links
}

# 🛑 Tắt toàn bộ Big Data stack
stop-bigdata() {
    echo "⚡ Đang tắt Spark..."
    (cd "~/hadoop-for-big-data-tool/docker-spark" && docker-compose down)
    
    echo "🐘 Đang tắt Hadoop + Hive + Hue..."
    (cd "~/hadoop-for-big-data-tool/docker-hadoop-hue-hive" && docker-compose down)
    
    echo "✅ Đã tắt toàn bộ dịch vụ."
}

# 📊 Hiển thị bảng thông tin dịch vụ
show-links() {
    echo ""
    echo "╔═══════════════════════════════════════════════════════════╗"
    echo "║           🎯 BẢNG ĐIỀU KHIỂN BIG DATA                     ║"
    echo "╠═══════════════════════════════════════════════════════════╣"
    printf "║ %-20s │ %-35s ║\n" "🔧 DỊCH VỤ" "🌍 ĐỊA CHỈ"
    echo "╠═══════════════════════════════════════════════════════════╣"
    printf "║ %-20s │ %-35s ║\n" "Hadoop NameNode" "http://localhost:50070"
    printf "║ %-20s │ %-35s ║\n" "Hadoop DataNode" "http://localhost:50075"
    printf "║ %-20s │ %-35s ║\n" "🎨 Hue Interface" "http://localhost:8888"
    printf "║ %-20s │ %-35s ║\n" "⚡ Spark Master" "http://localhost:8080"
    printf "║ %-20s │ %-35s ║\n" "Spark RPC" "spark://localhost:7077"
    printf "║ %-20s │ %-35s ║\n" "Hive JDBC" "jdbc:hive2://localhost:10000"
    echo "╚═══════════════════════════════════════════════════════════╝"
    echo "  💡 Đăng nhập Hue: admin/admin (hoặc tạo mới lần đầu)"
    echo "  🔄 Xem lại bảng này: show-links"
    echo ""
}

# 🔍 Kiểm tra nhanh trạng thái
bigdata-status() {
    echo "📊 Trạng thái Container:"
    docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" | grep -E "namenode|hive|hue|spark|datanode|resourcemanager"
}
```

Sau khi lưu file, áp dụng thay đổi:

```bash
source ~/.bashrc
```

---

## 📖 Hướng Dẫn Sử Dụng

### Các Lệnh Hàng Ngày

| Lệnh | Mô tả |
|------|-------|
| `start-bigdata` | 🚀 Khởi động tất cả dịch vụ |
| `stop-bigdata` | 🛑 Tắt tất cả dịch vụ |
| `show-links` | 📊 Hiển thị địa chỉ dịch vụ |
| `bigdata-status` | 🔍 Kiểm tra trạng thái container |

### Mô Tả Dịch Vụ

| Dịch vụ | Chức năng |
|---------|-----------|
| **🐘 Hadoop NameNode** | Quản lý HDFS - xem dung lượng, duyệt file |
| **🎨 Hue (8888)** | **Nơi làm việc chính** - soạn SQL, duyệt file, quản lý job |
| **⚡ Spark Master** | Giám sát Spark cluster, worker và ứng dụng đang chạy |
| **🐝 HiveServer2** | Thực thi truy vấn Hive qua kết nối JDBC |

---

## ⚠️ Lưu Ý Quan Trọng

### 🕐 Thời Gian Khởi Động
> Một số dịch vụ (Hive Metastore, Hue) cần **1-2 phút** sau khi container hiện "Up" để hoạt động đầy đủ. Nếu gặp lỗi, hãy chờ và thử lại.

### 📝 File Cấu Hình

| File | Vị trí | Mục đích |
|------|--------|----------|
| `hue-overrides.ini` | `docker-hadoop-hue-hive/` | Ghi đè cấu hình Hue |
| `hadoop-hive.env` | `docker-hadoop-hue-hive/` | Biến môi trường |
| `spark-defaults.conf` | `docker-spark/` | Cấu hình Spark |
| `hive-site.xml` | `docker-spark/` | Kết nối Hive cho Spark |

### 💾 Lưu Trữ Dữ Liệu
- Metadata của Hive được lưu trong PostgreSQL
- Dữ liệu HDFS được lưu trong Docker volumes
- **Cảnh báo**: Chạy `docker-compose down -v` sẽ xóa toàn bộ dữ liệu!

### 🔧 Xử Lý Sự Cố

| Vấn đề | Giải pháp |
|--------|-----------|
| Không tìm thấy network | Chạy `docker network create bigdata-net` |
| Dịch vụ không phản hồi | Chờ 2-3 phút, kiểm tra `docker logs <container>` |
| Cổng đã được sử dụng | Kiểm tra `netstat -tlnp \| grep <port>` và tắt dịch vụ xung đột |
| Hue bị khóa database | Restart container: `docker restart <hue-container>` |

---

## 📁 Cấu Trúc Dự Án

```
├── 📂 docker-hadoop-hue-hive/          # Stack Hadoop + Hive + Hue
│   ├── docker-compose.yml
│   ├── hadoop-hive.env
│   └── hue-overrides.ini
│
├── 📂 docker-spark/            # Spark cluster
│   ├── docker-compose.yml
│   ├── spark-defaults.conf
│   └── hive-site.xml
│
└── 📄 guide.md                 # Tài liệu này
```

---

## 📜 Giấy Phép

Dự án này phục vụ mục đích học tập.

---

**Được tạo với ❤️ cho việc học Big Data**
