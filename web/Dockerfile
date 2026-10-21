# Pinned so rebuilds are reproducible; bump deliberately (Debian 13 "trixie" base).
FROM nginx:1.31.7-trixie

# Python, Flask, PyQt5 (encoder.py imports QtWidgets) and uWSGI all come from
# Debian packages: Debian 12+ blocks system-wide pip installs (PEP 668), and this
# needs no compiler.
RUN apt-get update -y && \
    apt-get install -y --no-install-recommends \
      supervisor uwsgi uwsgi-plugin-python3 \
      python3 python3-flask python3-pyqt5 \
      procps libgl1 libglib2.0-0t64 && \
    rm -rf /var/lib/apt/lists/*

# Copy up cli side of the app
COPY ./src/ /app


# Source code file
COPY ./web/ /webapp
COPY ./web/node_modules/@forevolve/bootstrap-dark/dist /webapp/public/bootstrap

# Copy the configuration file from the current directory and paste
# it inside the container to use it as Nginx's default config.
COPY ./web/docker/nginx.conf /etc/nginx/nginx.conf

# setup NGINX config
RUN mkdir -p /spool/nginx /run/pid && \
    chmod -R 777 /var/log/nginx /var/cache/nginx /etc/nginx /var/run /run /run/pid /spool/nginx && \
    chgrp -R 0 /var/log/nginx /var/cache/nginx /etc/nginx /var/run /run /run/pid /spool/nginx && \
    chmod -R g+rwX /var/log/nginx /var/cache/nginx /etc/nginx /var/run /run /run/pid /spool/nginx && \
    rm /etc/nginx/conf.d/default.conf

# Copy the base uWSGI ini file to enable default dynamic uwsgi process number
COPY ./web/docker/uwsgi.ini /etc/uwsgi/apps-available/uwsgi.ini
RUN ln -s /etc/uwsgi/apps-available/uwsgi.ini /etc/uwsgi/apps-enabled/uwsgi.ini

COPY ./web/docker/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
RUN touch /var/log/supervisor/supervisord.log

EXPOSE 8000:8000

# setup entrypoint
COPY ./web/docker/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

# https://github.com/moby/moby/issues/31243#issuecomment-406879017
#RUN ln -s /usr/local/bin/docker-entrypoint.sh / && \
RUN chmod 777 /usr/local/bin/docker-entrypoint.sh && \
    chgrp -R 0 /usr/local/bin/docker-entrypoint.sh && \
    chown -R nginx:root /usr/local/bin/docker-entrypoint.sh

# https://docs.openshift.com/container-platform/3.3/creating_images/guidelines.html
RUN chgrp -R 0 /var/log /var/cache /run/pid /spool/nginx /var/run /run /tmp /etc/uwsgi /etc/nginx && \
    chmod -R g+rwX /var/log /var/cache /run/pid /spool/nginx /var/run /run /tmp /etc/uwsgi /etc/nginx && \
    chown -R nginx:root /webapp && \
    chmod -R 777 /webapp /etc/passwd

# enter
WORKDIR /webapp
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["supervisord"]
