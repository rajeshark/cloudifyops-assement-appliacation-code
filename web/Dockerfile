FROM node:19-alpine

WORKDIR /app

COPY package.json package-lock.json /app/

RUN npm install
COPY . /app
EXPOSE 8080

CMD ["npm", "run", "serve"]
