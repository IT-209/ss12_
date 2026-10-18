# Bài 2: Tự Động Hóa Đóng Gói Và Phát Hành Docker Image Khi Gắn Git Tag (v*..)

## 1. Kịch bản `.github/workflows/release.yml`

```yaml
name: Release Docker Image

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  release-image:
    name: Build & Push Docker Image
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v5

      - name: Log in to GitHub Container Registry (GHCR)
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata for Docker (tags, labels)
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=ref,event=tag

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

## 2. Bash kiểm tra & Minh chứng thực thi (thay thế ảnh chụp)

```bash
# 1. Gán git tag phiên bản và đẩy lên remote repo
git tag v1.0.0
git push origin v1.0.0

# 2. Theo dõi workflow release đang chạy
gh run list --workflow=release.yml --limit 1
gh run watch

# 3. Kiểm tra gói Docker Image v1.0.0 đã được phát hành trên GitHub Packages
gh api /orgs/${OWNER:-$(gh repo view --json owner -q .owner.login)}/packages/container/${REPO:-$(gh repo view --json name -q .name)}/versions \
  --jq '.[].metadata.container.tags[]' 2>/dev/null || \
gh api /user/packages/container/$(gh repo view --json name -q .name)/versions \
  --jq '.[].metadata.container.tags[]'

# 4. Kéo image về kiểm tra local (định dạng: ghcr.io/<owner>/<repo>:v1.0.0)
docker pull ghcr.io/$(git config --get remote.origin.url | sed -E 's/.*[:\/](.+)\/(.+)\.git/\1\/\2/'):v1.0.0
```
