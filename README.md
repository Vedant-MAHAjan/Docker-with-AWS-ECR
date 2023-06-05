# Docker with AWS ECR 

## Aim

Push a reduced Docker image of a React app to AWS Elastic Container Registry

## Keywords of the Dockerfile

- The ```FROM``` command specifies the base image for the build stage.
- The ```WORKDIR``` command sets the working directory for the build stage.
- The ```COPY``` command copies files from the local machine to the Docker image.
- The ```RUN``` command runs a command in the Docker image.

## How to run the Dockerfile...

Navigate to the directory containing the Dockerfile...

#### Run

```docker build -f Dockerfile.dev -t image_name:version_name .```

The version name can be anything like 1.0, version_1, etc.

## Explanation of the Dockerfile

There are 3 files with each file reducing the size of the final image.

### Dockerfile1
This is the basic Dockerfile with the maximum image size

```
FROM node
WORKDIR /app    
COPY package.json .   
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

- The base image is node with working directory as /app. 
The package.json file is copied to /app (this takes advantage of layer caching in Docker). The dependencies are installed and 
the rest of the files are copied to /app. 
The appropriate port 3000 is exposed (this command has no change on the final image, it is used just as information)
and the start the app 

## Dockerfile2
This file uses multi-stage builds. Since only build files are required to serve the site, 
all the other files are not copied into the final image thus reducing it's size.

```
# Stage 1
FROM node:12-alpine AS build
WORKDIR /app
COPY package.json .
RUN npm install
COPY . /app
RUN npm run build 

# Stage 2
FROM node:12-alpine
WORKDIR /app
RUN npm install -g webserver.local
COPY --from=build /app/build ./build
EXPOSE 3000
CMD webserver.local -d ./build
```
- The 1st stage or build stage creates a build folder inside the app directory after executing the build command. 
The "webserver.local" package is used to serve the built files

## Dockerfile3
This file uses Nginx to serve the site as it compatible and thus helps reduce the image size

```
# Stage1
FROM node:12-alpine AS build
WORKDIR /app
COPY package.json .
RUN npm install
COPY . /app
RUN npm run build 

# Stage2
FROM nginx:stable-alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
- The built files are copied to the ```/nginx/html``` directory and then served using nginx server



