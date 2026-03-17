# Making the Switch to Docker Hardened Images

Switching to a Docker Hardened Image is straightforward. All that is needed is to replace the base image `node:24-trixie-slim` with a DHI equivalent.

Most Docker Hardened Images come in two variants:

* Dev variant (`$$dhiPrefix$$node:24-debian13-dev`) – includes a shell and package managers, making it suitable for building and testing.
* Runtime variant (`$$dhiPrefix$$node:24-debian13`) – stripped down to only the essentials, providing a minimal and secure footprint for production.

Recognizing the _runtime variant_ doesn't have a shell or dependency manager (like `npm`), we need to use multi-stage Dockerfiles. We can build the app in the dev image, then copy the built application into the runtime image, which will serve as the base for production.

The idea is to follow this pattern:

1. Stage one will install the dependencies
2. Stage two will copy the dependencies from stage one and include the app code

## Writing the new Dockerfile

1. Update your :fileLink[Dockerfile]{path="Dockerfile"} to define a `build` stage using a DHI base image:

    ```dockerfile save-as=Dockerfile
    FROM $$dhiPrefix$$node:24-debian13-dev AS build

    ENV NODE_ENV=production
    WORKDIR /usr/src/app

    # Copy package files
    COPY package*.json ./

    # Install dependencies
    RUN npm i --omit=dev
    ```

2. Update the :fileLink[Dockerfile]{path="Dockerfile"} to define a `production` stage:

    ```dockerfile
    FROM $$dhiPrefix$$node:24-debian13 AS production
    ENV NODE_ENV=production
    WORKDIR /usr/src/app

    # Copy node_modules from the build stage
    COPY --from=build /usr/src/app/node_modules ./node_modules

    # Copy application code
    COPY src ./src
    COPY package*.json ./

    # Expose port (use non-privileged port)
    EXPOSE 3000

    # Start the application
    CMD ["node", "src/app.js"]
    ```


3. Build your new image with the following `docker build` command:

    ```bash
    docker build --provenance=true --sbom=true -t $$orgname$$/demo-node-dhi:v2 .
    ```

4. Validate the app still works by starting it:

   ```bash
   docker run -p 3050:3000 --name dhi-demo-app $$orgname$$/demo-node-dhi:v2
   ```

   And then open :tabLink[http://localhost:3050]{id="app" href="http://localhost:3050"}

4. Analyze the new image with `docker scout quickview`:

    ```bash
    docker scout quickview $$orgname$$/demo-node-dhi:v2
    ```

    You will see similar output:

    ```plaintext no-copy-button
      Target     │  orgname/demo-node-dhi:v2          │    0C     0H     0M     0L   
        digest   │  cec31e6f0a36                      │                              
      Base image │  orgname/dhi-node:24-debian13      │                              
    
    Policy status  SUCCESS  (9/9 policies met)
    
      Status │                   Policy                    │           Results            
    ─────────┼─────────────────────────────────────────────┼──────────────────────────────
      ✓      │ AGPL v3 licenses found                      │    0 packages                
      ✓      │ Default non-root user                       │                              
      ✓      │ No AGPL v3 licenses                         │    0 packages                
      ✓      │ No embedded secrets                         │    0 deviations              
      ✓      │ No embedded secrets (Rego)                  │    0 deviations              
      ✓      │ No fixable critical or high vulnerabilities │    0C     0H     0M     0L   
      ✓      │ No high-profile vulnerabilities             │    0C     0H     0M     0L   
      ✓      │ No unapproved base images                   │    0 deviations              
      ✓      │ Supply chain attestations                   │    0 deviations    
    ```

    🎉 Hooray! You should see several (if not all!) policies be met with the single change.

## Image comparison

Now that the image has been updated, you can look at the image size and package count advantages of using distroless Hardened Images.

Docker Scout offers a helpful command `docker scout compare` that allows you to analyze and compare two images. 

1. Compare the original and migrated image by using the following `docker scout compare` command:

    ```bash
    docker scout compare local://$$orgname$$/demo-node-dhi:v2 --to local://demo-node-doi:v1
    ```

    Scroll up the output and you will see a similar summary:

    ```plaintext no-copy-button
    ## Overview
      
                          │               Analyzed Image        │              Comparison Image                
      ────────────────────┼─────────────────────────────────────┼──────────────────────────────────────────────
        Target            │  local://orgname/demo-node-dhi:v2   │  local://demo-node-doi:v1   
          digest          │  39175f673409                       │  132bffffdbf5                                
          tag             │  latest                             │  v1                                          
          platform        │ linux/arm64                         │ linux/arm64                                  
          vulnerabilities │    0C     3H     0M     8L          │    0C     6H     1M    22L                   
                          │           -3     -1    -22          │                                              
          size            │ 42 MB (-39 MB)                      │ 82 MB  
          packages        │ 82 (-193)                           │ 275                                          
                          │                                     │                                              
        Base image        │  orgname/dhi-node:24                │  node:24-trixie-slim                         
          tags            │ also known as                       │ also known as                                
                          │                                     │   • 24.14-trixie-slim                     
                          │                                     │   • 24.14.1-trixie-slim  
                          |                                     |   • krypton-trixie-slim   
                          |                                     |   • lts-trixie-slim      
          vulnerabilities │    0C     0H     0M     0L          │    0C     6H     1M    22L                   
      
    ```

As you can see, the `dhi-node:24-debian13`–based image is **40 MB (around 40%) smaller**, contains **193 fewer packages**, and has nearly **zero CVEs** compared to the original `node:24-trixie-slim`–based image.
