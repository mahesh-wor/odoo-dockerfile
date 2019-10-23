FROM debian:stretch

MAINTAINER BI

# Define build constants
ENV GIT_BRANCH=7.0 \
  PYTHON_BIN=python3 \
  SERVICE_BIN=odoo-bin

# Set timezone to UTC
RUN ln -sf /usr/share/zoneinfo/Etc/UTC /etc/localtime

# Install APT dependencies
ADD sources/apt.txt /opt/sources/apt.txt
# locales
RUN apt-get update && apt install -yq node-less && apt-get install -y locales \
    && awk '! /^ *(#|$)/' /opt/sources/apt.txt | xargs -r apt  -yq install \
    && rm -rf /var/lib/apt/lists/* \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG en_US.utf8

# Generate locales


# RUN apt update \
#   # && apt install -yq curl \
#   # && curl -sL https://deb.nodesource.com/setup_10.x > setup_10.x && chmod +x setup_10.x && ./setup_10.x \
#   # && apt install -yq nodejs \
#   # && apt install -yq  build-essential libssl-dev && npm install -g less less-plugin-clean-css \
#   && awk '! /^ *(#|$)/' /opt/sources/apt.txt | xargs -r apt  -yq install
#   #&& ln -s /usr/bin/nodejs /usr/bin/node
#   # && npm install -g less less-plugin-clean-css

# Create the odoo user
RUN useradd --create-home --home-dir /opt/odoo --no-log-init odoo

# Switch to user odoo to create the folders mapped with volumes, else the
# corresponding folders will be created by root on the host
USER odoo

# If the folders are created with "RUN mkdir" command, they will belong to root
# instead of odoo! Hence the "RUN /bin/bash -c" trick.
RUN /bin/bash -c "mkdir -p /opt/odoo/{etc,sources/odoo,additional_addons,data,ssh}"

# Add Odoo sources and remove .git folder in order to reduce image size
WORKDIR /opt/odoo/sources
RUN git clone https://github.com/OCA/OCB -b $GIT_BRANCH --depth=1 \
  && mv OCB/* odoo && rm -rf odoo/.git OCB

ADD sources/odoo.conf /opt/odoo/etc/odoo.conf
ADD auto_addons /opt/odoo/auto_addons

User 0

# Install Odoo python dependencies
RUN pip install -r /opt/odoo/sources/odoo/requirements.txt

# Install extra python dependencies
ADD sources/pip.txt /opt/sources/pip.txt
RUN pip install -r /opt/sources/pip.txt

# Install wkhtmltopdf based on QT5
RUN curl -o wkhtmltox.tar.xz -SL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.4/wkhtmltox-0.12.4_linux-generic-amd64.tar.xz \
        && echo '3f923f425d345940089e44c1466f6408b9619562 wkhtmltox.tar.xz' | sha1sum -c - \
        && tar xvf wkhtmltox.tar.xz \
        && cp wkhtmltox/lib/* /usr/local/lib/ \
        && cp wkhtmltox/bin/* /usr/local/bin/ \
        && rm -rf wkhtmltox

# Startup script for custom setup
ADD sources/startup.sh /opt/scripts/startup.sh

#for readwrite odoo user.

RUN chmod -R 775 /opt/odoo && chown -R odoo:odoo /opt/odoo

VOLUME [ \
  "/opt/odoo/etc", \
  "/opt/odoo/additional_addons", \
  "/opt/odoo/data", \
  "/opt/odoo/ssh", \
  "/opt/scripts" \
]

RUN rm -rf /root/.cache/*  \
        && rm -rf /var/lib/apt/lists/* \
        && rm -rf /var/cache/apt/* \
        && apt -yq autoclean \
        && apt -yq autoremove \
        && apt remove -yq git \
        && rm -rf /root/.cache/* \
        && mkdir -p /mnt/extra-addons \
        && chown -R odoo /mnt/extra-addons


# RUN rm -rf /root/.cache/*
# RUN mkdir -p /mnt/extra-addons \
#         && chown -R odoo /mnt/extra-addons
# Expose the odoo ports (for linked containers)
EXPOSE 8069 8072
User odoo
# Set the default config file
ENV ODOO_RC /opt/odoo/etc/odoo.conf
ADD sources/entrypoint.sh /opt/scripts/entrypoint.sh
User 0
RUN chmod  775 /opt/scripts/entrypoint.sh && chown odoo:odoo /opt/scripts/entrypoint.sh
RUN ["chmod", "+x", "/opt/scripts/entrypoint.sh"]
User odoo
#RUN ["chmod", "+x", "/opt/scripts/entrypoint.sh"]
# USER odoo
ENTRYPOINT ["/opt/scripts/entrypoint.sh"]
CMD ["/opt/odoo/sources/odoo/odoo-bin"]
