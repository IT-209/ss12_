# Bài 1: CI Đa Phiên Bản JDK (Matrix Strategy) cho Java Spring Boot Gradle

## 1. Kịch bản `.github/workflows/ci-matrix.yml`

```yaml
name: CI Java Matrix Test

on:
  push:
    branches:
      - main

jobs:
  test-matrix:
    name: Test on Java ${{ matrix.java-version }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java-version: ['17', '21']
    steps:
      - name: Checkout repository
        uses: actions/checkout@v5

      - name: Setup Java ${{ matrix.java-version }}
        uses: actions/setup-java@v5
        with:
          java-version: ${{ matrix.java-version }}
          distribution: 'temurin'
          cache: 'gradle'

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Run unit tests with Gradle
        run: ./gradlew test
```

## 2. Bash kiểm tra & Minh chứng thực thi (thay thế ảnh chụp)

```bash
# 1. Push code lên nhánh main để kích hoạt workflow
git add .github/workflows/ci-matrix.yml
git commit -m "ci: add matrix test on java 17 and 21"
git push origin main

# 2. Kiểm tra trạng thái ma trận bằng GitHub CLI (hiển thị 2 sub-job song song)
gh run list --workflow=ci-matrix.yml --limit 1

# 3. Xem chi tiết kết quả 2 sub-jobs (17 và 21) đều thành công
gh run view --log
```
