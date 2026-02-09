FROM node:18.9.1-alpine

WORKDIR /app

# Install dependencies separately to leverage Docker cache
COPY package*.json ./
RUN npm install --production

# Copy source code
COPY . .

# Use non-root user for security
USER node

EXPOSE 5050

CMD ["node", "server.js"]

