# base Image
FROM node:20-alpine AS base
WORKDIR /usr/src/app

# install dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat
COPY package*.json . 
RUN npm install

# build the project
FROM base AS builder
COPY --from=deps /usr/src/app/node_modules node_modules
COPY . .
RUN npm run build

# development environment
FROM base AS dev
EXPOSE 3000
COPY --from=deps /usr/src/app/node_modules node_modules
COPY . .
ENV NODE_ENV development
CMD ["npm", "run", "dev"]

# prod environment
FROM base as prod
ENV NODE_ENV production

# create new user group
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Set the correct permission for prerender cache
RUN mkdir .next
RUN chown nextjs:nodejs .next

# only copy needed folders
COPY --from=builder  /usr/src/app/public ./public
COPY --from=builder --chown=nextjs:nodejs /usr/src/app/.next/static ./.next/static
COPY --from=builder --chown=nextjs:nodejs /usr/src/app/.next/standalone ./

# run as nextjs
USER nextjs

# env variables, can be overridden by docker compose
ENV HOSTNAME "0.0.0.0"
ENV PORT 3000
EXPOSE ${PORT}

CMD [ "node", "server.js" ]









