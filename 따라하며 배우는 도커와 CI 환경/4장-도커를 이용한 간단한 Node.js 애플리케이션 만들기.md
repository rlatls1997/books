# 4. 도커를 이용한 간단한 Node.js 애플리케이션 만들기

Dockerfile을 만들자
```dockerfile
FROM node:14

# 애플리케이션 디렉터리 생성
WORKDIR /usr/src/app

# 애플리케이션 종속성 설치하기
COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 8080

CMD ["node", "server.js"]
```

## 4.1 Node.js 애플리케이션 만들기
p.77참고