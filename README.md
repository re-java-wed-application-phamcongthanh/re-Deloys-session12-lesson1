# Bài 1: Xây Dựng Quy Trình CI Đa Phiên Bản JDK (Matrix Strategy) Cho Java Spring Boot Gradle

## 🎯 1. Mục Tiêu
- Cấu hình ma trận kiểm thử tự động (**Matrix Strategy**) trong GitHub Actions để chạy song song các bài test trên nhiều phiên bản JDK (**Java 17** và **Java 21**).
- Tối ưu hóa hiệu năng và thời gian thực thi ma trận bằng cơ chế **Gradle Dependency Caching** (`cache: 'gradle'`).
- Đảm bảo ứng dụng Spring Boot tương thích ngược và chạy ổn định trên các phiên bản môi trường thực thi (LTS Java Runtimes) khác nhau trước khi phát hành.

---

## 📁 2. Cấu Trúc Thư Mục
```text
bai1/
├── .github/
│   └── workflows/
│       └── ci-matrix.yml        # Kịch bản CI ma trận kiểm thử đa JDK
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── src/
│   ├── main/java/com/example/demo/
│   │   └── DemoApplication.java
│   └── test/java/com/example/demo/
│       └── DemoApplicationTests.java
├── .gitignore
├── build.gradle
├── gradlew                      # Executable script (chmod 755)
├── gradlew.bat
├── settings.gradle
└── README.md                    # Tài liệu kỹ thuật chi tiết
```

---

## ⚙️ 3. Kịch Bản Workflow Ma Trận (`.github/workflows/ci-matrix.yml`)

```yaml
name: Java Spring Boot Matrix CI (Gradle)

on:
  push:
    branches: [ "main" ]

jobs:
  test-matrix:
    name: Test on Java ${{ matrix.java-version }}
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        java-version: ['17', '21']
    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập Java JDK ${{ matrix.java-version }}
        uses: actions/setup-java@v5
        with:
          java-version: ${{ matrix.java-version }}
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test
```

### 💡 Phân Tích Kỹ Thuật Matrix Strategy:
1. **Từ khóa `strategy.matrix`**:
   - Cho phép định nghĩa một ma trận các biến môi trường (trong bài là `java-version: ['17', '21']`).
   - GitHub Actions sẽ tự động nhân bản (fan-out) Job `test-matrix` thành **2 sub-job độc lập chạy song song**:
     - Sub-job 1: `Test on Java 17`
     - Sub-job 2: `Test on Java 21`
2. **Cờ `fail-fast: false`**:
   - Mặc định nếu một sub-job lỗi, GitHub sẽ hủy bỏ các job còn lại. Đặt `fail-fast: false` đảm bảo cả 2 môi trường đều được kiểm thử đầy đủ để lập trình viên có cái nhìn toàn diện về tính tương thích.
3. **Bộ nhớ đệm `cache: 'gradle'`**:
   - Mỗi sub-job tự động tận dụng và chia sẻ cache Gradle dependencies, giúp rút ngắn thời gian chạy kiểm thử xuống dưới 20 giây cho mỗi phiên bản JDK.

---

## ✅ 4. Xác Nhận Kết Quả Trên GitHub Actions
1. **Đẩy mã nguồn lên GitHub**:
   ```bash
   git add .
   git commit -m "feat: complete session 12 exercise 1 with Java 17 & 21 matrix strategy"
   git push origin main
   ```
2. **Kiểm tra tab Actions**:
   - Truy cập: `https://github.com/re-java-wed-application-phamcongthanh/re-Deloys-session12-lesson1/actions`
   - Quan sát luồng chạy: Job tự động phân nhánh thành 2 job con:
     - `Test on Java 17` (Tích xanh ✅)
     - `Test on Java 21` (Tích xanh ✅)
3. Chụp màn hình luồng Matrix Test hiển thị 2 sub-jobs đều hoàn thành thành công để nộp bài.
