## 1. Tạo file Github Action

- .github/workflows/docker-image.yml

```bash
name: Docker Image CI

on:
  push:
    branches: ['main']
  pull_request:
    branches: ['main']

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: 'Create env file'
        run: echo "${{ secrets.ENV }}" > .env
      - name: Build the Docker image
        run: docker build --progress=plain -t {USERNAME}/{name-project}:latest .
      - name: Log in to Docker Hub
        run: docker login ghcr.io -u ${{ secrets.USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
      - name: Push the Docker image
        run: docker push ghcr.io/{USERNAME}/{name-project}:latest

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Executing remote ssh commands using password
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.HOST_USERNAME }}
          password: ${{ secrets.HOST_PASSWORD }}
          port: ${{ secrets.PORT }}
          script: |
            docker login ghcr.io -u ${{ secrets.USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
            docker pull ghcr.io/{USERNAME}/{name_project}:latest
            docker stop {name_deploy}
            docker rm {name_deploy}
            docker run -p 3000:3000  --name {name_deploy} --restart unless-stopped ghcr.io/{USERNAME}/{NAME_PROJECT}:latest

```

## 2. Tạo Secret and Variable

- Vào cài đặt của repo -> chọn Secret and Variables -> chọn action -> Tạo:

```bash
USERNAME là id của github
```

```bash
DOCKER_PASSWORD là Access Token từ github
```

```bash
HOST là địa chỉ ip vps
```

```bash
HOST_USERNAME là tên của host ví dụ hau@ip_vps
```

```bash
HOST_PASSWORD là mật khẩu của vps
```

```bash
ENV copy toàn bộ biến env vào
```
