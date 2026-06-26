# --- Etapa 1: Compilación ---
FROM node:20-alpine AS build
WORKDIR /app

# Copiar archivos de dependencias para instalarlas
COPY package*.json ./
RUN npm install

# Copiar el resto del código del frontend y compilarlo
COPY . .
RUN npm run build

# --- Etapa 2: Servidor Web para producción ---
FROM nginx:alpine

# Copiar los archivos estáticos compilados al directorio de Nginx
# NOTA: Si usas Vite o Angular, la carpeta de salida se llama 'dist'. 
# Si usas React clásico (Create React App), se suele llamar 'build'.
COPY --from=build /app/dist /usr/share/nginx/html

# Configuración básica para que las rutas de Single Page Apps (SPA) no den error 404 al recargar
RUN echo 'server { listen 80; location / { root /usr/share/nginx/html; index index.html; try_files $uri $uri/ /index.html; } }' > /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]