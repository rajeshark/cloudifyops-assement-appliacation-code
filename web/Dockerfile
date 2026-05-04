FROM node:16-alpine

# 1. Add this line to fix OpenSSL error
ENV NODE_OPTIONS=--openssl-legacy-provider

WORKDIR /app

# 2. Copy package files
COPY package*.json ./

# 3. Add --no-optional to skip optional dependencies
RUN npm install --no-optional

COPY . /app

EXPOSE 8080

CMD ["npm", "run", "serve"]
