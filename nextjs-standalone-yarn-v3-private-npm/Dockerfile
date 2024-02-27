FROM node:18-alpine AS base

FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json yarn.lock* package-lock.json* pnpm-lock.yaml* ./

ARG NPM_TOKEN=$NPM_TOKEN
ARG NPM_REGISTRY=$NPM_REGISTRY
ARG NPM_REGISTRY_HTTP=$NPM_REGISTRY_HTTP
ARG NPM_SCOPE=$NPM_SCOPE

RUN npm config set //${NPM_REGISTRY}:_authToken ${NPM_TOKEN} \
&& npm config set @${NPM_SCOPE}:registry https://${NPM_REGISTRY}

RUN yarn config set "npmRegistries['${NPM_REGISTRY_HTTP}'].npmAlwaysAuth" true \
&& yarn config set "npmRegistries['${NPM_REGISTRY_HTTP}'].npmAuthToken" ${NPM_TOKEN} \
&& yarn config set "npmScopes['${NPM_SCOPE}'].npmAlwaysAuth" true \
&& yarn config set "npmScopes['${NPM_SCOPE}'].npmRegistryServer" ${NPM_REGISTRY_HTTP} \
&& yarn config set nodeLinker node-modules

RUN yarn

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ARG STAGE=$STAGE
# if STAGE is set as "test", it will copy `.env.stage` to `.env.local`
COPY .env.$STAGE .env.local

RUN npm run build
ENV NEXT_SHARP_PATH=./node_modules/sharp

# Production image, copy all the files and run next
FROM base AS runner
WORKDIR /app
ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
COPY --from=builder /app/public ./public

RUN mkdir .next
RUN chown nextjs:nodejs .next
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]