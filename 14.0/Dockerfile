FROM debian:buster-slim
MAINTAINER Odoo S.A. <info@odoo.com>

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG C.UTF-8

# Install some deps, lessc and less-plugin-clean-css, and wkhtmltopdf
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        dirmngr \
        fonts-noto-cjk \
        gnupg \
        libssl-dev \
        node-less \
        npm \
	rustc \
        python3-num2words \
        python3-pdfminer \
        python3-pip \
        python3-phonenumbers \
        python3-pyldap \
        python3-qrcode \
        python3-renderpm \
        python3-setuptools \
        python3-slugify \
        python3-vobject \
        python3-watchdog \
        python3-xlrd \
        python3-xlwt \
        xz-utils \
        git \
        python3-venv \
        libpq-dev \
        python3-pypdf2 \
        wget \
        build-essential \
        python3-dev  \
        python3-wheel \
        libxslt1-dev \
        libzip-dev \
        libldap2-dev \
        libsasl2-dev \
        python3-setuptools \
        libpng-tools \
        libjpeg-dev \
        gdebi \
    && curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.buster_amd64.deb \
    && echo 'ea8277df4297afc507c61122f3c349af142f31e5 wkhtmltox.deb' | sha1sum -c - \
    && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
    && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# install latest postgresql-client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ buster-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
    && GNUPGHOME="$(mktemp -d)" \
    && export GNUPGHOME \
    && repokey='B97B0AFCAA1A47F044F244A07FCC7D46ACCC4CF8' \
    && gpg --batch --keyserver keyserver.ubuntu.com --recv-keys "${repokey}" \
    && gpg --batch --armor --export "${repokey}" > /etc/apt/trusted.gpg.d/pgdg.gpg.asc \
    && gpgconf --kill all \
    && rm -rf "$GNUPGHOME" \
    && apt-get update  \
    && apt-get install --no-install-recommends -y postgresql-client \
    && rm -f /etc/apt/sources.list.d/pgdg.list \
    && rm -rf /var/lib/apt/lists/*

# Install rtlcss (on Debian buster)
RUN npm install -g rtlcss


# Crear estructura de carpetas
RUN adduser --system --quiet --shell=/bin/bash --no-create-home --gecos 'ODOO' --group odoo
RUN mkdir -p /opt/odoo/extra-addons \
    && mkdir -p /opt/odoo/data \
    && mkdir -p /opt/odoo/src \
    && mkdir -p /opt/odoo/config \
    && chown -R odoo /opt/odoo

#Configurar Git
RUN git config --global user.name "Eladio Garcia"
RUN git config --global user.email eladio77@gmail.com

# Bajar fuentes de Odoo
ENV ODOO_VERSION 14.0
RUN git clone https://github.com/odoo/odoo.git -b $ODOO_VERSION --depth=1  /opt/odoo/src

# Expose Odoo services
EXPOSE 8069 8071 8072

# Install pip pack
ENV CRYPTOGRAPHY_DONT_BUILD_RUST=1
RUN pip3 install cryptography libsass geopy paramiko pyopenssl

# Instalar dependencias Odoo
RUN pip3 install -r /opt/odoo/src/requirements.txt 

# Set default user when running container
RUN ln -s /opt/odoo/src/odoo-bin /usr/bin/odoo
RUN chown -R odoo: /opt/odoo

# Copy entrypoint script and Odoo configuration file
COPY ./odoo.conf /opt/odoo/config
COPY ./entrypoint.sh /

# Set permissions and Mount /var/lib/odoo to allow restoring filestore and /mnt/extra-addons for users addons
RUN chown odoo /opt/odoo/config/odoo.conf 
VOLUME ["/opt/odoo/data","/opt/odoo/extra-addons"]

# Set default config file
ENV ODOO_RC /opt/odoo/config/odoo.conf
COPY wait-for-psql.py /usr/local/bin/wait-for-psql.py

# Set default user when running the container
USER odoo

ENTRYPOINT ["/entrypoint.sh"]
CMD ["odoo"]
