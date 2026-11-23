
# Stage 1: build the Angular app into static files (HTML/JS/CSS)
FROM node:16-alpine AS build
WORKDIR /app
COPY package.json ./
RUN npm install --force
COPY . .
RUN npm run build

# Stage 2: serve those files with Nginx
FROM nginx:1.27-alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY --from=build /app/dist/angular-app /usr/share/nginx/html
EXPOSE 80
