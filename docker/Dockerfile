FROM clojure:lein-alpine as builder

RUN apk --no-cache add \
    git \
    nodejs \
    nodejs-npm
RUN npm config set unsafe-perm true
RUN npm install -g bower stylus

WORKDIR /netrunner
COPY . .
RUN bower --allow-root install
RUN stylus src/css -o resources/public/css/
RUN lein deps
RUN lein cljsbuild once prod

# Run fetch here to import card data at build time since /admin/fetch doesn't seem to actually do the db import.
# - Env var MONGO_CONNECTION_URI must be defined in OCP BuildConfig otherwise this will fail with a db timeout.
# ie: mongodb://mongo:27017/netrunner
RUN lein fetch

RUN lein uberjar

FROM openjdk:8-alpine

WORKDIR /netrunner
COPY --from=builder /netrunner/target/netrunner-standalone.jar .
COPY --from=builder /netrunner/dev.edn .
WORKDIR /netrunner/resources
COPY --from=builder /netrunner/resources .
WORKDIR /netrunner/data
COPY --from=builder /netrunner/data .
WORKDIR /netrunner/src
COPY --from=builder /netrunner/src .
WORKDIR /netrunner
# make data writable for card fetches TODO: maybe not required
RUN chmod -R 777 /netrunner/data


ENTRYPOINT ["java", "-jar", "netrunner-standalone.jar"]
