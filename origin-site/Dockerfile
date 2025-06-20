FROM nginx:alpine

# Copy the static website files to nginx's serving directory
COPY . /usr/share/nginx/html/

# Remove default nginx configuration
RUN rm /etc/nginx/nginx.conf

# Copy our custom nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Expose port 80
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"] 