# Node.js Applications

[Back to guide index](README.md)

## 4.1 Node.js Deployment Model

Node.js apps commonly include:

- APIs with Express, Fastify, NestJS
- SSR apps like Next.js
- Static site builds
- Worker processes
- WebSocket servers

## 4.2 Node Version Management

### 4.2.1 nvm

`nvm` is widely used for per-user Node.js version management.

Install:

```bash
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
$ source ~/.nvm/nvm.sh
```

Use:

```bash
$ nvm install 20
$ nvm use 20
$ node -v
```

### 4.2.2 n

`n` is a simple Node version manager.

```bash
# npm install -g n
# n 20
```

### 4.2.3 fnm

`fnm` is a fast Node manager written in Rust.

Benefits:

- Very fast startup
- Good shell integration

## 4.3 Package Managers

Common choices:

- npm
- yarn
- pnpm

Comparison:

| Tool | Strengths | Notes |
|---|---|---|
| npm | Default, universal | Good default choice |
| yarn | Mature workspace support | Yarn classic vs berry differs |
| pnpm | Fast, disk-efficient | Excellent for monorepos |

## 4.4 `package.json` Scripts

Scripts standardize lifecycle commands.

Example:

```json
{
  "scripts": {
    "dev": "node server.js",
    "build": "vite build",
    "start": "node dist/server.js",
    "test": "vitest run"
  }
}
```

Use:

```bash
$ npm run build
$ npm run test
$ npm start
```

## 4.5 Dependency Installation

Use clean install in CI.

npm:

```bash
$ npm ci
```

Yarn:

```bash
$ yarn install --frozen-lockfile
```

pnpm:

```bash
$ pnpm install --frozen-lockfile
```

Why:

- Uses lockfile strictly
- More reproducible than floating installs

## 4.6 Build and Bundling Tools

Modern Node and frontend-oriented projects commonly use:

- webpack
- Vite
- esbuild
- Rollup

### 4.6.1 webpack

Powerful and highly configurable.

Use for:

- Complex frontend bundling
- Legacy asset pipelines
- Advanced plugin ecosystems

### 4.6.2 Vite

Fast developer experience and excellent modern frontend build support.

Use for:

- React
- Vue
- Svelte
- Static SPA builds

### 4.6.3 esbuild

Extremely fast bundler and transformer.

Use for:

- Tooling
- Fast builds
- Simple production bundles

## 4.7 Express Application Example

Basic app:

```javascript
const express = require('express');
const app = express();

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.listen(3000, '127.0.0.1', () => {
  console.log('Listening on 3000');
});
```

Install and run:

```bash
$ npm install express
$ node server.js
```

## 4.8 Building Node Apps

Typical production sequence:

```bash
$ npm ci
$ npm test
$ npm run build
```

For TypeScript:

```bash
$ npm ci
$ npm run lint
$ npm run test
$ npm run build
$ node dist/server.js
```

## 4.9 PM2 Process Manager

PM2 is widely used for long-running Node processes.

Install:

```bash
# npm install -g pm2
```

Start an app:

```bash
$ pm2 start dist/server.js --name mynodeapp
```

Useful commands:

```bash
$ pm2 status
$ pm2 logs mynodeapp
$ pm2 restart mynodeapp
$ pm2 stop mynodeapp
$ pm2 delete mynodeapp
$ pm2 save
```

Cluster mode example:

```bash
$ pm2 start dist/server.js --name mynodeapp -i max
```

## 4.10 PM2 Ecosystem File

Example `ecosystem.config.js`:

```javascript
module.exports = {
  apps: [
    {
      name: 'mynodeapp',
      script: './dist/server.js',
      instances: 2,
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3000
      }
    }
  ]
};
```

Run:

```bash
$ pm2 start ecosystem.config.js
```

## 4.11 systemd vs PM2

| Aspect | systemd | PM2 |
|---|---|---|
| OS-native | Yes | No |
| Good for non-Node services too | Yes | No |
| User-level Node convenience | Moderate | High |
| Centralized Linux operations | Excellent | Moderate |
| Cluster mode built-in | No | Yes |

For standardized Linux ops, systemd is usually preferred.

PM2 is convenient when teams are Node-centric.

## 4.12 Deploying Express Behind Nginx

Nginx config:

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 4.13 Deploying Next.js

There are two common patterns:

- `next start` behind Nginx
- Static export when the app is fully static

Build and run:

```bash
$ npm ci
$ npm run build
$ npm run start
```

If using standalone output, package the `.next/standalone` directory.

Example systemd `ExecStart`:

```ini
ExecStart=/usr/bin/node /opt/nextapp/current/server.js
```

## 4.14 Node.js systemd Service

Example service file:

```ini
[Unit]
Description=Node.js API Service
After=network.target

[Service]
Type=simple
User=nodeapp
Group=nodeapp
WorkingDirectory=/opt/nodeapp/current
EnvironmentFile=/etc/nodeapp/nodeapp.env
ExecStart=/usr/bin/node /opt/nodeapp/current/dist/server.js
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
```

Example env file:

```bash
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://nodeapp:secret@db.internal/nodeapp
```

## 4.15 Node Build Artifacts

Common build outputs:

- `dist/`
- `.next/`
- `build/`
- `public/` assets
- Transpiled server bundle

Keep production deployments minimal.

Do not copy:

- Tests
- Local caches
- Dev-only assets
- Unnecessary source maps unless needed for debugging

## 4.16 Lockfiles Matter

Use exactly one lockfile for a project.

Examples:

- `package-lock.json`
- `yarn.lock`
- `pnpm-lock.yaml`

Avoid mixing package managers.

## 4.17 Node Deployment Layout

```text
/opt/nodeapp/
├── current -> /opt/nodeapp/releases/2025-01-15_120000/
├── releases/
└── shared/
    ├── logs/
    └── uploads/
```

## 4.18 WebSocket Proxying for Node Apps

Nginx config example:

```nginx
location /socket.io/ {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
}
```

## 4.19 Common Node Deployment Mistakes

- Running `npm install` instead of `npm ci` in CI
- Forgetting to build TypeScript before starting
- Missing reverse proxy WebSocket headers
- Using dev dependencies at runtime unintentionally
- Running as root
- Not setting `NODE_ENV=production`

## 4.20 Production Node Checklist

- Correct Node version installed
- Lockfile present and used
- Build output generated
- Reverse proxy configured
- Health endpoint exposed
- Memory limits monitored
- Logs centralized
- Restart policy enabled

---
