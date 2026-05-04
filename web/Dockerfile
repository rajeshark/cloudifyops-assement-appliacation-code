FROM node:16-alpine

WORKDIR /app

COPY package*.json ./

# Remove the NODE_OPTIONS line - Node 16 doesn't need it!
RUN npm install

COPY . /app

EXPOSE 8080

CMD ["npm", "run", "serve"]
