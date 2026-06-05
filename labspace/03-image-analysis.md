# Image analysis

## Exploring the app

This demo repository contains a simple Node.js application consisting of a basic JS server. The app is implemented in the :fileLink[src/app.js]{path="src/app.js"} file. 

1. Build the app with the `docker build` command:

    ```bash
    docker build -t demo-node-doi:v1 .
    ```

2. Run the app using the following `docker run` command:

    ```bash
    docker run -p 3050:3000 --name dhi-demo-app demo-node-doi:v1
    ```

3. Open the app by going to :tabLink[http://localhost:3050]{href="http://localhost:3050" id="app" title="App" icon="draft"}. You should see a simple "Hello World!" message.

4. Stop the app with the following `docker rm` command:

    ```bash
    docker rm -f dhi-demo-app
    ```

## Build and analyze the container image

To prepare this application for production, **you should ensure the image is secure by using [Docker Scout](https://www.docker.com/products/docker-scout/).**

1. Build the image using the following `docker build` command, this time adding a [SBOM](https://docs.docker.com/build/metadata/attestations/sbom/) and build provenance:

    ```bash
    docker build --provenance=true --sbom=true -t $$orgname$$/demo-node-doi:v1 .
    ```

    The `--provenance=true` and `--sbom=true` flags are added to attach [build attestations](https://docs.docker.com/build/metadata/attestations/) to the image, which Docker Scout uses to provide more detailed and accurate security analysis.


2. Analyze the image by using the `docker scout quickview` command:

    ```bash
    docker scout quickview $$orgname$$/demo-node-doi:v1
    ```

    You will see similar output:

    ```plaintext no-copy-button
     Target      │  orgname/demo-node-doi:v1           │    0C     2H     2M    20L   
        digest   │  771a1b07daa3                       │                              
      Base image │  node:24-trixie-slim                │    0C     1H     2M    20L   
    
    Policy status  FAILED  (6/10 policies met)
    
      Status │                              Policy                              │           Results            
    ─────────┼──────────────────────────────────────────────────────────────────┼──────────────────────────────
      ✓      │ AGPL v3 licenses found                                           │    0 packages                
      !      │ No default non-root user found                                   │                              
      ✓      │ No AGPL v3 licenses                                              │    0 packages                
      ✓      │ No embedded secrets                                              │    0 deviations              
      ✓      │ No embedded secrets (Rego)                                       │    0 deviations              
      !      │ Fixable critical or high vulnerabilities found                   │    0C     2H     0M     0L   
      ✓      │ No high-profile vulnerabilities                                  │    0C     0H     0M     0L   
      !      │ Unapproved base images found                                     │    1 deviation               
      ✓      │ Supply chain attestations                                        │    0 deviations              
    ```

    As you can see, there are no CVEs at the application level, but the base image contains a number of high, medium, and low severity CVEs, so it is recommended to be updated. 
    
    Additionally, the critical policies have failed:

    1. No default non-root user found
    2. Fixable critical or high vulnerabilities found
    3. Unapproved base images found
 
This is where DHI comes into play.

