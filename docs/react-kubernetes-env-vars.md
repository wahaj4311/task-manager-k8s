# Runtime Environment Variables in React with Kubernetes

## The Challenge with React and Environment Variables

React applications typically handle environment variables at build time through the `process.env` object. This creates a challenge when running React apps in containers and Kubernetes because:

1. Environment variables are "baked into" the JavaScript bundle during build
2. Changes to environment variables require rebuilding the application
3. The same image cannot be easily reused across different environments

## The Solution: Runtime Configuration Injection

This pattern allows you to:
- Build the image once
- Run it in different environments (dev, staging, prod)
- Change environment variables without rebuilding
- Use the same image across all environments

### Implementation Steps

1. **Create an Environment Script (`env.sh`)**
```bash
#!/bin/sh

# Recreate config file
echo "window._env_ = {" > /usr/share/nginx/html/env-config.js
echo "  REACT_APP_API_URL: \"$REACT_APP_API_URL\"," >> /usr/share/nginx/html/env-config.js
echo "  REACT_APP_ENV: \"$REACT_APP_ENV\"" >> /usr/share/nginx/html/env-config.js
echo "}" >> /usr/share/nginx/html/env-config.js
```

2. **Update Dockerfile**
```dockerfile
FROM node:16-alpine as builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
COPY env.sh /docker-entrypoint.d/40-env.sh
RUN chmod +x /docker-entrypoint.d/40-env.sh
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

3. **Add Script to HTML (`public/index.html`)**
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <script src="%PUBLIC_URL%/env-config.js"></script>
    <!-- other head elements -->
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

4. **Update React Code to Use Runtime Config**
```javascript
function App() {
  // Use runtime config with fallback to build-time env vars
  const API_URL = window._env_?.REACT_APP_API_URL || 
                 process.env.REACT_APP_API_URL || 
                 'http://localhost:5000';
  
  // Rest of your component code
}
```

5. **Create Kubernetes ConfigMap**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-config
data:
  REACT_APP_API_URL: "http://api-service:5000"
  REACT_APP_ENV: "production"
```

6. **Update Kubernetes Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  template:
    spec:
      containers:
      - name: frontend
        image: your-frontend-image:tag
        env:
        - name: REACT_APP_API_URL
          valueFrom:
            configMapKeyRef:
              name: frontend-config
              key: REACT_APP_API_URL
```

## How It Works

1. When the container starts:
   - The `env.sh` script runs
   - Creates/updates `env-config.js` with current environment variables
   - Makes variables available through `window._env_`

2. When the React app loads:
   - Loads `env-config.js` before the application code
   - Makes environment variables available at runtime
   - App uses `window._env_` to access current configuration

## Benefits

1. **Build Once, Run Anywhere**
   - Same image works in all environments
   - No need to rebuild for config changes

2. **Easy Configuration Updates**
   - Update ConfigMap and restart pods
   - No image rebuilding required

3. **Environment Separation**
   - Different configs for dev/staging/prod
   - Same image, different settings

4. **Kubernetes Native**
   - Works well with Kubernetes ConfigMaps
   - Follows container best practices

## Best Practices

1. Always provide fallback values for critical configuration
2. Use TypeScript interfaces for environment configuration
3. Document required environment variables
4. Include validation for required variables
5. Use meaningful prefixes (e.g., `REACT_APP_`) for clarity

## Common Issues and Solutions

1. **Environment Variables Not Updating**
   - Ensure pods are restarted after ConfigMap changes
   - Check if `env.sh` has execute permissions

2. **Script Not Running**
   - Verify script location in Dockerfile
   - Check container logs for script execution

3. **Variables Undefined**
   - Check ConfigMap is properly mounted
   - Verify environment variable names match exactly 