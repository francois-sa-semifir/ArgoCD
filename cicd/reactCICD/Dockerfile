# Build stage
FROM node:20 as build
WORKDIR /app
COPY . .
RUN npm install -g pnpm
RUN pnpm install && pnpm run build

# Production stage
FROM nginx:stable-alpine as production
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]