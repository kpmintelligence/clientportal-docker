#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#

FROM laradock/workspace:1.0

MAINTAINER Janos Budai <janos.budai@fathomminds.com>

#
#--------------------------------------------------------------------------
# Mandatory Software's Installation
#--------------------------------------------------------------------------
#
# Mandatory Software's such as ("php7.1-cli", "git", "vim", ....) are
# installed on the base image 'laradock/workspace' image. If you want
# to add more Software's or remove existing one, you need to edit the
# base image (https://github.com/LaraDock/workspace).
#

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# Optional Software's will only be installed if you set them to `true`
# in the `docker-compose.yml` before the build.
#
#   - INSTALL_XDEBUG=           false
#   - INSTALL_MONGO=            false
#   - COMPOSER_GLOBAL_INSTALL=  false
#   - INSTALL_NODE=             false
#

#####################################
# xDebug:
#####################################

# Check if xDebug needs to be installed
ARG INSTALL_XDEBUG=true
ENV INSTALL_XDEBUG ${INSTALL_XDEBUG}
RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    # Load the xdebug extension only with phpunit commands
    apt-get install -y --force-yes php7.1-xdebug && \
    sed -i 's/^/;/g' /etc/php/7.1/cli/conf.d/20-xdebug.ini && \
    echo "alias phpunit='php -dzend_extension=xdebug.so /var/www/laravel/vendor/bin/phpunit'" >> ~/.bashrc \
;fi

#####################################
# MongoDB:
#####################################

# Check if Mongo needs to be installed
ARG INSTALL_MONGO=true
ENV INSTALL_MONGO ${INSTALL_MONGO}
RUN if [ ${INSTALL_MONGO} = true ]; then \
    # Install the mongodb extension
    pecl install mongodb && \
    echo "extension=mongodb.so" >> /etc/php/7.0/cli/php.ini \
;fi

#####################################
# Non-Root User:
#####################################

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ARG PGID=1000
RUN groupadd -g $PGID laradock && \
    useradd -u $PUID -g laradock -m laradock
    
# Now switch to our laradock user for the rest of user setup
USER laradock

#####################################
# Composer:
#####################################

# Add the composer.json
ADD ./composer.json /home/laradock/.composer/composer.json

# Check if global install need to be runned
ARG COMPOSER_GLOBAL_INSTALL=true
ENV COMPOSER_GLOBAL_INSTALL ${COMPOSER_GLOBAL_INSTALL}
RUN if [ ${COMPOSER_GLOBAL_INSTALL} = true ]; then \
    # run the install
    composer global install \
;fi

#####################################
# Node / NVM:
#####################################

# Install NVM
USER root
RUN mkdir /usr/local/nvm
ENV NVM_DIR /usr/local/nvm
ENV NODE_VERSION 7.4.0
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.3/install.sh | bash
RUN . $NVM_DIR/nvm.sh && \
    nvm install $NODE_VERSION && \
    nvm use $NODE_VERSION && \
    nvm alias $NODE_VERSION && \
    npm install -g gulp bower webpack@1.14.0
ENV PATH $PATH:/usr/local/nvm/versions/node/v7.4.0/bin/npm
RUN ln -s /usr/local/nvm/versions/node/v7.4.0/bin/npm /usr/local/bin/npm
RUN ln -s /usr/local/nvm/versions/node/v7.4.0/bin/node /usr/local/bin/node
RUN ln -s /usr/local/nvm/versions/node/v7.4.0/bin/webpack /usr/local/bin/webpack

RUN echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \

# Wouldn't execute when added to the RUN statement in the above block
# Source NVM when loading bash since ~/.profile isn't loaded on non-login shell
RUN if [ ${INSTALL_NODE} = true ]; then \
    echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc \
;fi

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

# Clean up
USER root
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Set default work directory
WORKDIR /var/www/laravel
