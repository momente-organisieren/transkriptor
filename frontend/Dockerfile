FROM nginx:alpine

# Nginx Konfiguration für SPA und API-Proxy
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Frontend Dateien
COPY index.html /usr/share/nginx/html/
COPY app.js /usr/share/nginx/html/
COPY styles.css /usr/share/nginx/html/
COPY logo.png /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
