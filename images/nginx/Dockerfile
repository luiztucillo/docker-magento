FROM nginx:latest

ARG username
ENV USERNAME=$username

ARG domain
ENV DOMAIN=$domain

ARG cert_path
ENV CERT_PATH=$cert_path

ARG cert_key_path
ENV CERT_KEY_PATH=$cert_key_path

#set folder owner
RUN mkdir -p /var/www/html
RUN useradd -m $USERNAME
RUN chown -R $USERNAME:$USERNAME /var/www/html

# Make sure the volume mount point is empty
RUN rm -rf /var/www/html/*

COPY conf/nginx.conf /etc/nginx/

RUN echo "set \$username $USERNAME;\n" >> /etc/nginx/var.conf
RUN echo "set \$domain $DOMAIN;\n" >> /etc/nginx/var.conf
