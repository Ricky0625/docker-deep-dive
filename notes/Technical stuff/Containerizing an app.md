# Containerizing an app - The deep dive

## The TDLR

Containers are all about making apps simple to **build**, **ship**, and **run**. The end-to-end process looks like this:

1. Start with your application code and dependencies
2. Create a *Dockerfile* that describes your app, dependencies, and how to run it
3. Build it into an image by passing the Dockerfile to `docker build` command
4. Push the new image to a registry (optional)
5. Run a container from the image

## Containerize a single-container app

The high level steps:

- Clone the repo to get the app code
- Inspect the Dockerfile
- Containerize the app
- Run the app
- Test the app
- Look a bit closer

### Getting the application code

This can be done either cloning the repo from a remote repository or you have your source code locally.

### Inspecting the Dockerfile

- A Dockerfile describes an application and tells Docker how to build it into an image.
- Dockerfile is a great form of documentation. It helps bridging the gap between developers and operations.

Example of a Dockerfile:

```Dockerfile
FROM alpine
LABEL maintainer="xxxx@gmail.com"
RUN apk add --update nodejs npm
COPY . /src
WORKDIR /src
RUN npm install
EXPOSE 8080
ENTRYPOINT ["node", "./app.js"]
```

At a high level, the example Dockerfile says:

1. Start with alpine image (base image), make a note that "xxxx@gmail.com" is the maintainer.
2. Install node.js and npm.
3. Copy everything in the build context (the directory where the Dockerfile is located) to the `/src` directory in the image.
4. Set working directory as `/src`
5. Install dependencies
6. Document the app's network port
7. Set `app.js` as the default application to run

Look a little bit deeper:

1. `FROM` - Dockerfile usually start with this instruction. This pulls an image that will be used as the base layer for the image the Dockerfile will build. Everything else will be added as new layers above this base layer.
2. `LABEL` - Optional. Add custom metadata. It's considered a best practice to list a maintainer so that other users have a point of contact to report problems.
3. `RUN` - Create a new layer in the Docker image. It is mainly used to set up environment variables, install software packages. It's the comman passed to run exectues on top of the current image in a new layer.
4. `COPY` - Copies folder or files onto a new layer.
5. `WORKDIR` - Set the working directory for the rest of the instructions. This creates metadata and does not create a new image layer.
6. `EXPOSE` - Make a specific port accessible to other services within the same network. Unlike `PORTS`, `EXPOSE` doesn't map to the host machine's ports. Instead, it establishes a communication line among the containers.

### Containerize the  app/build the image

- To build a new image based on a Dockerfile, we can do something like this:

	```shell
	docker build -t theapp:v0.1 .
	```

- The period `.` at the end of the command tells Docker to use the working directory as the *build context*.
- `docker build` with `-t` means that you want to name and optionally assign a tag for that image
- Build context is where the app and all dependencies are stored.

### Run the app

Once you have successfully built the image, you can create a container and run it. The following command can be used:

```shell
docker run -d --name c1 -p 80:8080 ddd-book:ch8.1
```

- `-d`: detach mode
- `--name`: name the container
- `-p`: map port

If your host is already running a service on port `80`, you will get a **port is already allocated** error. If this happens, specify a different port such as `5000` or `5001`, just any other port.

### Looking a bit closer

- The `docker build` command parses the Dockerfile one-line-at-a-time starting from the top.
- All non-comment lines are **Instructions** and take the format `<INSTRUCTION> <arguments>`.
- Instruction names are not case sensitive but it's normal practice to write them in UPPERCASE to make reading the file easier.
- Some instructions create new layers whereas others just add metadata.
- Instructions that create new layers: `FROM`, `RUN`, `COPY`.
- Instructions that create metadata: `EXPOSE`, `WORKDIR`, `ENV`, `ENTRYPOINT`.
- The basic premise is this: If an instruction adds content such as files and programs, it will create a new layer. If it is adding instructions on how to build the image and run the container, it will create metadata.
- View the instructions that were used to build the image with the `docker history` command.

> There may be a bug in the builder used by Docker that causes the `WORKDIR` instruction to create a layer.

###  Moving to production with Multi-stage Builds

- For Docker images, *big is bad*! Big means slow, means more potential vulnerabilities, means a bigger attack surface.
- Container images should only contain the stuff needed to run your app in production.
- Multi-stage builds make keeping images small easy.
- Multi-stage builds have multiple `FROM` instructions in a single Dockerfile, and each `FROM` instruction is a new **build stage**.

Example:

```Dockerfile
FROM golang:1.20-alpine AS base
WORKDIR /src
COPY go.mod go.sum .
RUN go mod download
COPY . .

FROM base AS build-client
RUN go build -o /bin/client ./cmd/client

FROM base AS build-server
RUN go build -o /bin/server ./cmd/server

FROM scratch AS prod
COPY --from=build-client /bin/client /bin/
COPY --from=build-server /bin/server /bin/
ENTRYPOINT [ "/bin/server" ]
```

This Dockerfile is used to build a Go application with separate stages for building the client and server components, and then creates a minimal production image using the `scratch` base image.

Here's a summary of the Dockerfile:

1. **Base Stage (`base`):**
   - Uses the official `golang:1.20-alpine` image as the base.
   - Sets the working directory to `/src`.
   - Copies `go.mod` and `go.sum` files and runs `go mod download` to download the dependencies.
   - Copies the entire application code into the container.

2. **Build Client Stage (`build-client`):**
   - Uses the `base` stage as the base image.
   - Builds the Go application for the client and places the binary at `/bin/client`.

3. **Build Server Stage (`build-server`):**
   - Also uses the `base` stage as the base image.
   - Builds the Go application for the server and places the binary at `/bin/server`.

4. **Production Stage (`prod`):**
   - Uses the `scratch` image as the base, providing a minimal and empty file system.
   - Copies the built client and server binaries from the previous build stages into the `/bin/` directory.
   - Configures the entry point to execute the server binary (`/bin/server`) when the container starts.

This Dockerfile employs multi-stage builds to keep the final production image small by only including the necessary artifacts. The resulting image (`prod`) will only contain the compiled binaries without any unnecessary dependencies from the build stages.

### Multi-stage builds and build targets

- It's possible to build multiple images from a single Dockerfile.

Example:

```Dockerfile
FROM golang:1.20-alpine AS base
WORKDIR /src
COPY go.mod go.sum .
RUN go mod download
COPY . .

FROM base AS build-client
RUN go build -o /bin/client ./cmd/client

FROM base AS build-server
RUN go build -o /bin/server ./cmd/server

FROM scratch AS prod-client
COPY --from=build-client /bin/client /bin/
ENTRYPOINT [ "/bin/client" ]

FROM scratch AS prod-server
COPY --from=build-server /bin/server /bin/
ENTRYPOINT [ "/bin/server" ]
```

This Dockerfile is also using multi-stage builds to create separate production images for the client and server components of a Go application. Each component is built in its own stage, and the final production images are based on the `scratch` image to keep them minimal.

Here's a summary of the Dockerfile:

1. **Base Stage (`base`):**
   - Uses the official `golang:1.20-alpine` image as the base.
   - Sets the working directory to `/src`.
   - Copies `go.mod` and `go.sum` files and runs `go mod download`.
   - Copies the entire application code into the container.

2. **Build Client Stage (`build-client`):**
   - Uses the `base` stage as the base image.
   - Builds the Go application for the client and places the binary at `/bin/client`.

3. **Build Server Stage (`build-server`):**
   - Also uses the `base` stage as the base image.
   - Builds the Go application for the server and places the binary at `/bin/server`.

4. **Production Client Stage (`prod-client`):**
   - Uses the `scratch` image as the base, providing a minimal file system.
   - Copies the built client binary from the `build-client` stage to `/bin/`.
   - Configures the entry point to execute the client binary (`/bin/client`) when the container starts.

5. **Production Server Stage (`prod-server`):**
   - Uses the `scratch` image as the base.
   - Copies the built server binary from the `build-server` stage to `/bin/`.
   - Configures the entry point to execute the server binary (`/bin/server`) when the container starts.

With this setup, you can create separate Docker images for the client and server components of your Go application, each with its own minimal file system.

```shell
docker build -t multi:client --target prod-client -f Dockerfile-final .
docker build -t multi:server --target prod-server -f Dockerfile-final .
```
