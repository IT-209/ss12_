# Bài 3: Tách Biệt Luồng CI/CD Đa Môi Trường (Staging & Production) Với GitHub Environments

## 1. Cấu hình GitHub Environments (Giao diện GitHub Settings)

1. Vào **Settings** -> **Environments** -> Nhấn **New environment**.
2. Tạo 2 môi trường:
   - `staging`: Không cần reviewer.
   - `production`: Tích chọn **Required reviewers**, thêm tài khoản GitHub của bạn làm người phê duyệt.

## 2. Kịch bản `.github/workflows/cd-multi-env.yml`

```yaml
name: Multi-Environment CD

on:
  push:
    branches:
      - develop
      - main

jobs:
  deploy-staging:
    name: Deploy to Staging
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - name: Checkout code
        uses: actions/checkout@v5

      - name: Deploying to Staging Environment
        run: |
          echo "Starting deployment to Staging environment..."
          echo "Deploying branch: ${{ github.ref_name }} (Commit: ${{ github.sha }})"
          echo "Deployment to Staging completed successfully."

  deploy-prod:
    name: Deploy to Production
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://api.prod.example.com
    steps:
      - name: Checkout code
        uses: actions/checkout@v5

      - name: Deploying to Production Environment
        run: |
          echo "Starting deployment to Production environment..."
          echo "Deploying branch: ${{ github.ref_name }} (Commit: ${{ github.sha }})"
          echo "Deployment to Production completed successfully."
```

## 3. Bash kiểm tra & Minh chứng thực thi (thay thế ảnh chụp)

```bash
# --- Kiểm tra Staging (Tự động chạy không cần duyệt) ---
git checkout -b develop 2>/dev/null || git checkout develop
git push origin develop
gh run list --workflow=cd-multi-env.yml --limit 1

# --- Kiểm tra Production (Chờ phê duyệt manual review) ---
git checkout main
git push origin main

# Lấy run ID mới nhất của nhánh main
RUN_ID=$(gh run list --workflow=cd-multi-env.yml --branch main --json databaseId -q '.[0].databaseId')

# Kiểm tra trạng thái đang chờ review (Status: waiting)
gh run view $RUN_ID

# (Optional) Thực hiện phê duyệt deployment trực tiếp qua GitHub CLI
# gh run review $RUN_ID --approve -c "Approved for production release"
```
