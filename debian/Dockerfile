FROM node:6.13.0

# Build arguments to change source url, branch or tag
ARG HACKMD_REPOSITORY=https://github.com/getstation/hackmd.git
ARG VERSION=master

# Set some default config variables
ENV DEBIAN_FRONTEND noninteractive
ENV DOCKERIZE_VERSION v0.3.0
ENV NODE_ENV=production

RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz && \
    tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz && \
    rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

# Add configuraton files
COPY config.json .sequelizerc /files/

RUN apt-get update && \
    apt-get install -y git build-essential && \

    # Clone the source
    git clone --depth 1 --branch $VERSION $HACKMD_REPOSITORY /hackmd && \
    # Print the cloned version and clean up git files
    cd /hackmd && \
    git log --pretty=format:'%ad %h %d' --abbrev-commit --date=short -1 && echo && \
    rm -rf /hackmd/.git && \

    # Symlink configuration files
    rm -f /hackmd/config.json && ln -s /files/config.json /hackmd/config.json && \
    rm -f /hackmd/.sequelizerc && ln -s /files/.sequelizerc /hackmd/.sequelizerc && \

    # Install NPM dependencies and build project
    yarn install --pure-lockfile && \
    yarn install --production=false --pure-lockfile && \
    yarn global add webpack && \
    npm run build && \

    # Clean up this layer
    npm prune --production && \
    yarn cache clean && \
    npm cache clean && \
    apt-get remove -y --auto-remove build-essential && \
    apt-get clean && apt-get purge && rm -r /var/lib/apt/lists/* && \
    # Create hackmd user
    adduser --uid 10000 --home /hackmd/ --disabled-password --system hackmd && \
    chown -R hackmd /hackmd/

WORKDIR /hackmd
EXPOSE 3000
USER hackmd

COPY docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]
