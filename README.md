# CircleCI Runner Setup

Repository with various files to install CircleCI's runner on Kubernetes via Helm chart.

## Prerequisites
- You must be on our [Scale Plan](https://circleci.com/pricing/) or sign up for a trial. [Reach out to our sales team](https://circleci.com/contact-us/?cloud) to ask about both.
- [Generate a token and resource class](https://circleci.com/docs/2.0/runner-installation/?section=executors-and-images#authentication) for your runner. For each different type of runner you want to run, you will need to repeat these same steps.
  - For example, if you want ten runners that pull the same types of jobs or run the same [parallel job](https://circleci.com/docs/2.0/parallelism-faster-jobs/) based on availability, you only need to create one runner resource class. All ten runners would share the same token.
  - If you want to run ten separate runners that pull different jobs that do different things, we recommend creating ten different runner resource classes. Each one would have a different name and use a different token.
- Have a Kubernetes cluster you would like to install the runner(s) in.

## Setup
1. Clone this repository.
2. Modify values as needed in `values.yaml`:

Value             | Description                  | Default
------------------|------------------------------|-------------
image.repository<br />image.tag | You can [extend a custom Docker image](https://circleci.com/docs/2.0/runner-installation/?section=executors-and-images#create-a-dockerfile-that-extends-the-circleci-runner-image) from the CircleCI default runner and use that instead. | `circleci/runner`<br />`launch-agent`
resource_class    | The resource class you created for your runner. We recommend not inserting it into `values.yaml` directly and setting it when you install your chart instead. See next step. | ""
runner_token      | The token you created for your runner. We recommend not inserting it into `values.yaml` directly and setting it when you install your chart instead. See next step. | ""
All other values  | Modify at your own discretion and risk. | N/A

3. Using the resource class name and token you created in the [Prerequisites](#prerequisites) section, you'll want to set parameters as you install the Helm chart:

```bash
$ helm install "circleci-runner" ./ \
  --set runner_token=$CIRCLECI_RUNNER_TOKEN \
  --set resource_class=$CIRCLECI_RUNNER_RESOURCE_CLASS \
  --namespace your-namespace
```
4. Call your runner class(es) in your job(s). Example:

```yaml
version: 2.1

executors:
  my-runner:
    machine: true
    resource_class: my-namespace/my-runner-resource-class
  
workflows:
  my-workflow:
    jobs:
      - my-job

jobs:
  my-job:
    executor: my-runner
    steps:
      - checkout
      - run: echo "Hello from my custom runner!"
```

## Known Issues/Pending Work
- Autoscaling is not yet implemented - for now, you'll need to manually modify the `replicaCount` in `values.yaml` and update the cluster.
