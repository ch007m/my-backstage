# This dockerfile builds an image for the backend package.
# It should be executed with the root of the repo as docker context.
#
# Before building this image, be sure to have run the following commands in the repo root:
#
# yarn install
# yarn tsc
# yarn build
#
# Once the commands have been run, you can build the image using `yarn build-image`

FROM node:16-bullseye-slim

WORKDIR /app

# install sqlite3 dependencies, you can skip this if you don't use sqlite3 in the image
RUN apt-get update && \
    apt-get install -y --no-install-recommends libsqlite3-dev python3 pip build-essential curl openjdk-11-jdk graphviz fontconfig && \
    rm -rf /var/lib/apt/lists/* && \
    yarn config set python /usr/bin/python3

# Download plantuml file, Validate checksum & Move plantuml file
RUN curl -o plantuml.jar -L http://sourceforge.net/projects/plantuml/files/plantuml.1.2022.4.jar/download && echo "246d1ed561ebbcac14b2798b45712a9d018024c0  plantuml.jar" | sha1sum -c - && mv plantuml.jar /opt/plantuml.jar

RUN pip install mkdocs-techdocs-core==1.0.2

# Create script to call plantuml.jar from a location in path
#   When adding TechDocs to the Backstage Backend container, avoid this
#   error (OSError: [Errno 8] Exec format error: 'plantuml') by using the
#   following RUN command instead:
#   RUN echo '#!/bin/sh\n\njava -jar '/opt/plantuml.jar' ${@}' >> /usr/local/bin/plantuml
RUN echo $'#!/bin/sh\n\njava -jar '/opt/plantuml.jar' ${@}' >> /usr/local/bin/plantuml
RUN chmod 755 /usr/local/bin/plantuml

# Copy repo skeleton first, to avoid unnecessary docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN yarn install --frozen-lockfile --production --network-timeout 300000 && rm -rf "$(yarn cache dir)"

# Then copy the rest of the backend bundle, along with any other files we might want.
COPY packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

CMD ["node", "packages/backend", "--config", "app-config.yaml", "--config", "app-config.production.yaml"]
