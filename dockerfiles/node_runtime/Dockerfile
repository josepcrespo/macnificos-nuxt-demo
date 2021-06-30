FROM node:lts-alpine

# Set the destination directory
WORKDIR /usr/src/app

# Copy the app files
COPY . /usr/src/app

# Install the `nmp` packages
RUN npm install

# Expose the app on the port 3000
EXPOSE 3000

# Run the `npm run dev` command
CMD ["npm", "run", "dev"]