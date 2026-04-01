# Stage 1: Build/Prepare stage
FROM alpine:latest AS build
WORKDIR /app
# Copy only the web files (adjust these folder names if your HTML is in a subfolder)
COPY index.html ./
COPY css/ ./css/
COPY js/ ./js/
COPY images/ ./images/

# Stage 2: Production stage
FROM nginx:alpine
# Copy ONLY the prepared files from the build stage
COPY --from=build /app /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
