FROM node:10.19.0

WORKDIR /app

COPY . .

RUN npm install body-parser cross-env express mocha mysql2 nodemon sequelize should sqlite3 supertest cors

EXPOSE 4006

CMD ["npm", "start"]

