FROM gcr.io/google.com/cloudsdktool/google-cloud-cli:slim

# Install nodejs LTS
RUN curl -fsSL https://deb.nodesource.com/setup_lts.x | bash -
RUN apt-get update && apt-get install -y nodejs wget apt-transport-https gnupg lsb-release \
    && rm -rf /var/lib/apt/lists/*

# Install tools, ex. postgresql-client, netcat
RUN apt-get update && apt-get install -y postgresql-client netcat \
    && rm -rf /var/lib/apt/lists/*

# Change to non-privilege user
USER cloudsdk

# Create and change to the app directory.
WORKDIR /home/cloudsdk

# Copy application dependency manifests to the container image.
# A wildcard is used to ensure both package.json AND package-lock.json are copied.
# Copying this separately prevents re-running npm install on every code change.
COPY package.json package*.json ./

# Install production dependencies.
RUN npm install --only=production

# Copy local code to the container image.
COPY . .

# Run the web service on container startup.
CMD [ "npm", "start" ]