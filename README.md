<div align="center">
  <img referrerpolicy="no-referrer-when-downgrade" src="https://static.scarf.sh/a.png?x-pxid=0fcbab94-8fbe-4a38-93e8-c2348450a42e" />
  <h1 align="center">ZenML Cloud Infrastructure Setup</h1>
</div>

<div align="center">
  <a href="https://zenml.io">
    <img alt="ZenML Logo" src="https://raw.githubusercontent.com/zenml-io/zenml/main/docs/book/.gitbook/assets/header.png" alt="ZenML Logo">
  </a>
  <br />
</div>

---

## ⭐️ Show Your Support

If you find this project helpful, please consider giving ZenML a star on GitHub. Your support helps promote the project and lets others know it's worth checking out.

Thank you for your support! 🌟

[![Star this project](https://img.shields.io/github/stars/zenml-io/zenml?style=social)](https://github.com/zenml-io/zenml/stargazers)

## 🚀 Overview

This Terraform module sets up the necessary GCP infrastructure for a [ZenML](https://zenml.io) stack. It provisions various GCP services and resources, and registers [a ZenML stack](https://docs.zenml.io/user-guide/production-guide/understand-stacks) using these resources with your ZenML server, allowing you to create an internal MLOps platform for your entire machine learning team.

## 🛠 Prerequisites

- Terraform installed (version >= 1.9")
- GCP account set up
- To authenticate with GCP, you need to have [the `gcloud` CLI](https://cloud.google.com/sdk/gcloud)
installed on your machine and you need to have run `gcloud init` or `gcloud auth login`
to set up your credentials.
- [ZenML (version >= 0.62.0) installed and configured](https://docs.zenml.io/getting-started/installation). You'll need a Zenml server deployed in a remote setting where it can be accessed from GCP. You have the option to either [self-host a ZenML server](https://docs.zenml.io/getting-started/deploying-zenml) or [register for a free ZenML Pro account](https://cloud.zenml.io/signup).

## 🏗 GCP Resources Created

The Terraform module in this repository creates the following resources in your GCP project:

1. a GCS bucket
2. a Google Artifact Registry
3. a Cloud Composer environment (only if the `orchestrator` variable is set to `airflow`)
4. a Service Account with the minimum necessary permissions to access the GCS bucket, the Google Artifact Registry and the GCP project to build and push container images with Google Cloud Build, store artifacts and run pipelines with Vertex AI, SkyPilot or GCP Cloud Composer.
5. depending on the target ZenML Server capabilities, different authentication methods are used:
  * for a self-hosted ZenML server, a Service Account Key is generated and shared with the ZenML server
  * for a ZenML Pro account, GCP Workload Identity Federation is used to authenticate with the ZenML server, so that no sensitive credentials are shared with the ZenML server. For this, a GCP Workload Identity Pool and a GCP Workload Identity Provider are created and linked to the GCP Service Account. There's only one exception: when the SkyPilot orchestrator is used, this authentication method is not supported, so the Service Account Key is used instead.

## 🧩 ZenML Stack Components

The Terraform module automatically registers a fully functional GCP [ZenML stack](https://docs.zenml.io/user-guide/production-guide/understand-stacks) directly with your ZenML server. The ZenML stack is based on the provisioned GCP resources and is ready to be used to run machine learning pipelines.

The ZenML stack configuration is the following:

1. an GCP Artifact Store linked to the GCS bucket
2. an GCP Container Registry linked to the Google Artifact Registry
3. depending on the `orchestrator` input variable:
  * a local Orchestrator, if `orchestrator` is set to `local`. This can be used in combination with the Vertex AI Step Operator to selectively run some steps locally and some on Vertex AI.
  * a Vertex AI Orchestrator linked to the GCP project, if `orchestrator` is set to `vertex` (default)
  * a SkyPilot Orchestrator linked to the GCP project, if `orchestrator` is set to `skypilot`
  * an Airflow Orchestrator linked to the Cloud Composer environment, if `orchestrator` is set to `airflow`
4. a Google Cloud Build Image Builder linked to the GCP project
5. a Vertex AI Step Operator linked to the GCP project
6. a GCP Service Connector configured with the GCP service account credentials or the GCP Workload Identity Provider configuration and used to authenticate all ZenML components with the GCP resources

To use the ZenML stack, you will need to install the required integrations:

* for Vertex AI:

```shell
zenml integration install gcp
```

* for SkyPilot:

```shell
zenml integration install gcp skypilot_gcp
```

* for Airflow:

```shell
zenml integration install gcp airflow
```


## 🚀 Usage

To use this module, aside from the prerequisites mentioned above, you also need to create [a ZenML Service Account API key](https://docs.zenml.io/how-to/connecting-to-zenml/connect-with-a-service-account) for your ZenML Server. You can do this by running the following command in a terminal where you have the ZenML CLI installed:

```bash
zenml service-account create <service-account-name>
```

### Basic Configuration

```hcl
module "zenml_stack" {
  source  = "zenml-io/zenml-stack/gcp"

  project_id = "your-gcp-project-id"
  region = "europe-west1"
  orchestrator = "vertex" # or "skypilot" or "airflow"
  zenml_server_url = "https://your-zenml-server-url.com"
  zenml_api_key = "ZENKEY_1234567890..."
}
output "zenml_stack_id" {
  value = module.zenml_stack.zenml_stack_id
}
output "zenml_stack_name" {
  value = module.zenml_stack.zenml_stack_name
}
```

## 🎓 Learning Resources

[ZenML Documentation](https://docs.zenml.io/)
[ZenML Starter Guide](https://docs.zenml.io/user-guide/starter-guide)
[ZenML Examples](https://github.com/zenml-io/zenml/tree/main/examples)
[ZenML Blog](https://www.zenml.io/blog)

## 🆘 Getting Help
If you need assistance, join our Slack community or open an issue on our GitHub repo.


<div>
<p align="left">
    <div align="left">
      Join our <a href="https://zenml.io/slack" target="_blank">
      <img width="18" src="https://cdn3.iconfinder.com/data/icons/logos-and-brands-adobe/512/306_Slack-512.png" alt="Slack"/>
    <b>Slack Community</b> </a> and be part of the ZenML family.
    </div>
    <br />
    <a href="https://zenml.io/features">Features</a>
    ·
    <a href="https://zenml.io/roadmap">Roadmap</a>
    ·
    <a href="https://github.com/zenml-io/zenml/issues">Report Bug</a>
    ·
    <a href="https://zenml.io/cloud">Sign up for ZenML Pro</a>
    ·
    <a href="https://www.zenml.io/blog">Read Blog</a>
    ·
    <a href="https://github.com/zenml-io/zenml/issues?q=is%3Aopen+is%3Aissue+archived%3Afalse+label%3A%22good+first+issue%22">Contribute to Open Source</a>
    ·
    <a href="https://github.com/zenml-io/zenml-projects">Projects Showcase</a>
  </p>
</div>
