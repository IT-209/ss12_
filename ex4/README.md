# Bài 4: Tự Động Triển Khai Ứng Dụng Spring Boot (Container) Lên VPS Từ Xa Qua SSH

## 1. Cấu hình Repository Secrets trên GitHub

Vào **Settings** -> **Secrets and variables** -> **Actions** -> **New repository secret**:
- `SSH_HOST`: Địa chỉ IP của máy chủ VPS (ví dụ: `203.0.113.10`).
- `SSH_USER`: Tên tài khoản SSH trên VPS (ví dụ: `root` hoặc `ubuntu`).
- `SSH_KEY`: Nội dung Private Key (`~/.ssh/id_rsa` hoặc `id_ed25519`).

## 2. Kịch bản `.github/workflows/deploy-vps.yml`

```yaml
name: Deploy Spring Boot Container to VPS

on:
  push:
    branches:
      - main

jobs:
  build-image:
    name: Build & Push Docker Image
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v5

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and Push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.actor }}/my-app:latest

  deploy-vps:
    name: Deploy to Remote VPS via SSH
    needs: build-image
    runs-on: ubuntu-latest
    steps:
      - name: Execute deployment commands on VPS via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
            docker pull ghcr.io/${{ github.actor }}/my-app:latest
            docker stop my-app || true
            docker rm my-app || true
            docker run -d --name my-app -p 8080:8080 ghcr.io/${{ github.actor }}/my-app:latest
            docker ps --filter "name=my-app"
```

## 3. Bash kiểm tra & Minh chứng thực thi (thay thế ảnh chụp)

```bash
# 1. Đẩy code lên main để kích hoạt build và deploy
git add .
git commit -m "ci/cd: build and deploy to remote VPS via SSH"
git push origin main

# 2. Xem logs triển khai SSH trực tiếp qua GitHub CLI
gh run watch
gh run view --log --job="Deploy to Remote VPS via SSH"

# 3. Kiểm tra trạng thái ứng dụng trên VPS từ terminal (xác nhận HTTP 200 OK)
curl -Is http://<VPS_IP>:8080/actuator/health | head -n 5
```
