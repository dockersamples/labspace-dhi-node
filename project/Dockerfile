FROM node:24-trixie-slim AS dev

ENV BLUEBIRD_WARNINGS=0
ENV NODE_ENV=production
ENV NODE_NO_WARNINGS=1
ENV NPM_CONFIG_LOGLEVEL=warn
ENV SUPPRESS_NO_CONFIG_WARNING=true

WORKDIR /app

COPY package* ./

RUN npm install --no-optional \
 && npm cache clean --force

COPY src ./src
EXPOSE 3000
CMD ["node", "src/app.js"]
