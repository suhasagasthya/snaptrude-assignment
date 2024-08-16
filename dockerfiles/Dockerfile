FROM node:20.3.0-bullseye-slim AS builder
ADD . /app
WORKDIR /app
RUN npm install --omit=dev

FROM "suhasagasthya/dockerhub:v1.0.0-nodejs-base"
COPY --from=builder /app /app
EXPOSE 6041 
CMD ["index.js"]
