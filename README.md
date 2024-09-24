# Docker-Image-Optimization

<h2 style="font-size: 25px;"> TEAM 👨‍👨‍👧 <br>
</h2>

|<img src="https://avatars.githubusercontent.com/u/81280628?v=4" width="100" height="100"/>|<img src="https://avatars.githubusercontent.com/u/86951396?v=4" width="100" height="100"/>
|:-:|:-:|
|[@손대현](https://github.com/DaeHyeonSon)|[@이아영](https://github.com/ayleeee)|
---

### 개요 🚩
Docker 이미지를 최적화를 위한 방법론에 대해 분석한 뒤 예제를 통해 최적화를 진행해보고자 한다.

### 작가의 의도 파악 🙄 -> [Reference]
Docker 이미지 최적화는 애플리케이션의 총비용, 효율성, 보안성 및 유지보수성을 향상시키는 중요한 실천사항이다. 이러한 체계적인 최적화 기법의 적용, 지속적인 개선, 팀과의 협업을 통해 개발자들이 보다 견고하고 성능이 우수한 배포를 실현할 수 있다.

<br>

> 1. 최소 기본 이미지 선택 :
가볍고 필수 구성 요소만 포함하여 이미지 크기가 확연하게 줄어드는 것을 확인 

<details>
<summary>예시</summary>
<div markdown="1">

```dockerfile
# Nginx의 경량 버전 사용
FROM nginx:alpine
```

</div>
</details>


> 2. 단일 책임 원칙 :
여러 서비스를 하나의 이미지에 포함하지 않는 하나의 서비스에 집중한 Docker 이미지 사용

<details>
<summary>예시</summary>
<div markdown="1">

```yaml
# docker-compose.yml 예시
version: '3'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
  app:
    image: myapp:latest
    build:
      context: ./app
```

</div>
</details>

> 3. muti-stage 빌드 사용 :
단일 Dockerfile에서 여러 FROM 문을 사용 가능 -> 최종 이미지에서 빌드 시간 종속성 및 아티팩트를 제거 가능 -> 최종 이미지 크기 줄어듬

<details>
<summary>예시</summary>
<div markdown="1">

```dockerfile
# Dockerfile 예시
# Build stage
FROM node:14-alpine as build

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:14-alpine

WORKDIR /app
COPY --from=build /app/package*.json ./
RUN npm ci --production
COPY --from=build /app/dist ./dist

CMD ["npm", "start"]
```

</div>
</details>

> 4. Layer 최소화 :
단일 run 명령어를 통해 Docker 이미지 레이어 수를 줄임 -> 이미지 크기 줄고, 빌드 프로세스 속도 증가

<details>
<summary>예시</summary>
<div markdown="1">

```dockerfile
RUN apt-get update && \
    apt-get install -y git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

</div>
</details>

> 5. .dockerignore 사용 :
.dockerignore을 사용해 docker 빌드 컨텍스트에서 불필요한 파일 및 dir 제외

<details>
<summary>예시</summary>
<div markdown="1">

```plaintext
# .dockerignore 예시
node_modules
*.log
.git
Dockerfile
docker-compose.yml
```

</div>
</details>

> 6. tag 사용 :
latest 태그 대신 특정 버전 태그를 사용하여 재현성을 보장하고 예상 외의 변수를 차단

<details>
<summary>예시</summary>
<div markdown="1">

```dockerfile
FROM nginx:1.21.6-alpine
```

</div>
</details>

> 7. Dockerfile 명령 최적화 :
패키지 설치 시 특정 버전을 사용

<details>
<summary>예시</summary>
<div markdown="1">

```dockerfile
RUN apt-get update && \
    apt-get install -y curl=7.68.0-1ubuntu2 && \
    apt-get purge -y --auto-remove && \
    rm -rf /var/lib/apt/lists/*
```

</div>
</details>

> 8. 아티팩트 압축 :
빌드 아티팩트를 압축하여 이미지 크기와 빌드 속도 개선

<details>
<summary>예시</summary>
<div markdown="1">

```bash
# 빌드 후 압축
RUN tar -czf dist.tar.gz dist
COPY dist.tar.gz /app/
RUN tar -xzf dist.tar.gz && rm dist.tar.gz
```

</div>
</details>

> 9.이미지 layer 검사 :
docker history와 docker inspect 도구를 사용하여 레이어를 분석하고 최적화 기회 식별

<details>
<summary>예시</summary>
<div markdown="1">

```bash
# 이미지 히스토리 확인
docker history myimage:latest

# 이미지 상세 정보 확인
docker inspect myimage:latest
```

</div>
</details>

> 10. Docker 이미지 정리 :
docker system prune 명령을 정기적으로 사용하여 사용하지 않는 이미지와 리소스 정리

<details>
<summary>예시</summary>
<div markdown="1">

```bash
# 모든 사용하지 않는 리소스 제거
docker system prune -a
```

</div>
</details>

> 11. 캐싱 구현
캐시 활용도를 최대화하는 방식으로 Dockerfile을 구성하여 Docker의 빌드 캐시 활용

<details>
<summary>예시</summary>
<div markdown="1">

```dockerfile
# 캐시 효율을 위한 Dockerfile 구조 예시
FROM node:14-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

CMD ["npm", "start"]
```

</div>
</details>

> 12. 보안 scanning
Docker 보안 검색 도구를 활용하여 Docker 이미지의 보안 취약점을 식별한 뒤 수정

<details>
<summary>예시</summary>
<div markdown="1">

```bash
# Docker Hub의 보안 스캔 사용
docker scan myimage:latest

# Trivy를 사용한 보안 스캔
trivy image myimage:latest
```

</div>
</details>

> 13. 더 작은 대안 사용
BusyBox나 Microcontainers와 같은 경량 도구 및 라이브러리를 사용

<details>
<summary>예시</summary>
<div markdown="1">

```dockerfile
# BusyBox 사용 예시
FROM busybox
COPY myapp /myapp
CMD ["/myapp"]
```

</div>
</details>

> 14. 설치 후 정리
패키지 설치 중 생성된 임시 파일 및 캐시를 제거하여 이미지 크기 줄임

<details>
<summary>예시</summary>
<div markdown="1">

```dockerfile
RUN apt-get update && \
    apt-get install -y build-essential && \
    make && \
    make install && \
    apt-get remove -y build-essential && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

</div>
</details>

> 15. Docker Squash 사용
layer를 병합하여 이미지 크기 줄임 BUT 빌드 시간이 늘어나고 캐시 가능성이 줄어들 수 있으므로 주의

<details>
<summary>예시</summary>
<div markdown="1">

```bash
# Docker Squash 설치
docker buildx create --use
docker buildx build --squash -t myimage:latest .
```

</div>
</details>

<hr>

### Reference 🙄
https://overcast.blog/docker-image-optimization-tips-tricks-6a17f687162b

*본 실습은 위 자료를 참조하여 제작하였습니다.*

