# Setup

Before running this lab, a few setup tasks are required.


## 1. 👥 Org setup

To help with command completion, please specify the name of your Docker org.

::variableDefinition[orgname]{prompt="What is the name of your Docker org?"}

:::conditionalDisplay{variable="tier" requiredValue="paid"}
:::conditionalDisplay{variable="orgname" hasNoValue}
> [!NOTE]
> Since you are wanting to use the paid-tier in this lab, please provide the name of your org.
:::

## 2. ✅ DHI Tier Selection

DHI offers both a **free tier** and a **paid tier**. The free tier images are available using the `dhi.io` registry while paid tier images are mirrored in your organization on Docker Hub.

Making a selection here ensures the commands throughout this lab work properly for you.

> [!TIP]
> **How to choose?** Use the free tier unless you have both a paid plan and have the [`dhi.io/node`](https://hub.docker.com/hardened-images/catalog/dhi/node) image mirrored in your organization.

::variableSetButton[Use the free tier]{variables="tier=free,dhiPrefix=dhi.io/"}

::variableSetButton[Use the paid tier ($$orgname$$)]{variables="tier=paid,dhiPrefix=$$orgname$$/dhi-"}


## 3. 🐳 Login with Docker 

1. In order to use Docker Scout to analyze the image during this lab, you will need to be logged in. Make sure that you are logged in with Docker:

    ```bash
    docker login
    ```

    After logging in, you should see the following message:

    ```bash no-run-button no-copy-button
    Login Succeeded
    ```

:::conditionalDisplay{variable="tier" requiredValue="free"}
2. In order to use the free tier, you will also need to log in with the `dhi.io` registry:

    ```bash
    docker login dhi.io
    ```
:::

## 4. 📝 Configure Docker Scout organization 

If your account is part of a paid organization, you may have additional output that reflects policy alignment.

```bash
docker scout config organization $$orgname$$
```

To learn more about Policy Evaluation in Docker Scout configure organization Policies, please follow [these guides](https://docs.docker.com/scout/policy/).
