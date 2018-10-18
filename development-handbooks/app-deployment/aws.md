# AWS

## Glossary

- **Elastic Cloud (EC2)** - Servers in AWS.
- **Identity Access Management (IAM)** - Permissions management.
- **Elastic Container Service (ECS)** - ECS allows you to
run a fleet of instances that are arranged as a flexible
`docker` cluster. You can add or remove instances
as you like.
- **Elastic Container Registry (ECR)** - ECR is nothing more
than a docker registry. For every app, you create one and
you can then put your tagged docker images into them.
- **Elastic Load Balancer (ELB)** - Load balancer in AWS, we
are using Application Load Balancer.
- **Fargate** - Runs service on AWS managed cluster. More
expensive, but you don't have to take care of EC2 instances.
- **Virtual Private Cloud (VPC)** - Virtual private network segment.

## How to create the environment

Ideally this is not really necessary, because you can deploy
your app into a shared cluster that's already running.

1. Start new ECS cluster. Do not use Fargate (for now).
We are using `t3.small` instances. Pick any size you need,
you can change it later. As for scaling, you should be
fine with a single instance to start with. Don't forget to
adjust the scaling rules (Autoscaling group) later. For your VPC,
use mulitple subnets.
1. Start an Application Load Balancer. This is required when
you run multiple services from your cluster - i. e. you need
routing. Set its Security group to accept only HTTP and HTTPS
inbound connections. **ELB has to be in the same VPC as your cluster.**
It will create a default target group, we'll change it later.
If you want HTTPS only (and you should), set up a default routing
rule for HTTP in a way that it redirects all traffic to port 443.
1. Set Security group to your cluster in a way that it accepts
All traffic, but *only* from a Security group that your ELB is in.

ELB gets connected to ECS cluster via Target groups. Target group is
basically a pipe that has a ELB on one side and containers on the other.
You will setup a separate target group for every service that runs in
your cluster.


## How to add a new service

A Service is always based on a Task. Task is like a template for a service.
We will be using a separate task for every container. This means that
`one application = one task = one service` in our case. After the following
guide, you should end up with a service (`wt-demo-app`) running from AWS ECS
on `demo.windingtree.com`.

But you can, for example, use tasks to compose whole environments composed
from many services.

To run a new version of a service on a commit, you need to do:

1. Build docker images.
1. Start a new version of a service.
1. Setup proper networking to your service.

### Building Docker image(s) on a release

This describes how to make Travis build and push docker images on tagged
builds (tagged commits).

#### Preconditions

- IAM user specific for Travis, let's call it `travis`. You will need
`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. You can use a single
IAM user for multiple services. One user can have multiple `ACCESS_KEY_ID`s.

#### AWS setup

1. Create new Repository in ECS called `wt-demo-app`.
1. Create new IAM policy so Travis can push to the repository. Don't forget
to change the Resource ID (it's on a Repository detail in ECS).
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecr:CompleteLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:InitiateLayerUpload",
                "ecr:BatchCheckLayerAvailability",
                "ecr:PutImage"
            ],
            "Resource": "arn:aws:ecr:region:xxxxxx:repository/wt-demo-app"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "ecr:GetAuthorizationToken",
            "Resource": "*"
        }
    ]
}
```
1. Assign that policy to `travis` user.

#### Travis setup

1. Add `AWS_ACCESS_KEY_ID` of `travis` AWS user to your Travis setup.
1. Add `AWS_SECRET_ACCESS_KEY` of `travis` AWS user to your Travis setup.
1. Tell Travis to build and push docker images on tagged builds. If you need
to differentiate between environments during build time, you need to build
multiple images here and use for example a suffix, such as `-playground`
in this instance. `$TRAVIS_BRANCH` will get replaced by an actual git tag.
Be careful about the addresses to your repository and AWS region.
```yaml
  - stage: Build docker image and upload to AWS
    sudo: true
    services:
    - docker
     install: true
    if: tag IS present
     script:
    - pip install --user awscli # install aws tools
    - export PATH=$PATH:$HOME/.local/bin # expose aws tools
    - eval $(aws ecr get-login --no-include-email --region region) # pick credentials for travis user
    - echo "Building for playground"
    - docker build --build-arg WT_CONFIG=playground -t wt-demo-app:$TRAVIS_BRANCH-playground .
    - docker tag wt-demo-app:$TRAVIS_BRANCH-playground xxxxxx.dkr.ecr.region.amazonaws.com/wt-demo-app:$TRAVIS_BRANCH-playground
    - docker push xxxxxx.dkr.ecr.region.amazonaws.com/wt-demo-app:$TRAVIS_BRANCH-playground
```
If you successfully set everything up and push a tag now, you should
see a new tagged image in your AWS ECR.

### Deploying new version of a service

This describes how to make Travis start a new version of your service
after merging a tag to `release/playground` branch.

#### Preconditions

- ECS Cluster.
- At least one deployed image in ECR.
- IAM user specific for Travis, let's call it `travis`. You will need
`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`. You can use a single
IAM user for multiple services. One user can have multiple `ACCESS_KEY_ID`s.

#### AWS setup

1. Create new IAM policy so Travis can work with tasks and services.
This is required only once, not for every service. If you are re-using
`travis` user for multiple services, you don't need to do this every time.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecs:UpdateService",
                "ecs:RegisterTaskDefinition"
            ],
            "Resource": "*"
        }
    ]
}
```
1. Assign that policy to `travis` user.
1. Create new Task definition in ECS. It's EC2 type. Pick a name
(we'll use `playground-wt-demo-app`) and keep the defaults. The important
part is `Container definitions`. Add a single container. The important bits
there are:
  - Image: You point it to your previously uploaded image in ECR. With a tag.
  - Port mappings: If your docker exposes port 3000, you should put in
  `Host port: 0` and `Container port: 3000`. 0 means that ports will
  be assigned dynamically.
  - Memory limits: This is different for every app, you can start with
  `Soft limit: 128`.
  - CPU units: That depends on your usage. You might reserve some CPU time for
  your app.
  - Env variables: These are used during `docker run` command.
1. Create a service. On task detail, pick the latest revision and select `Create
service` from Actions. There:
  - Type: EC2
  - Service name: In our case `playground-wt-demo-app`. In our setup, it might be
  wise to keep it in line with the task.
  - Number of tasks: 1 - This means that one instance will always run.
  - Load balancer: Select Application Load Balancer and choose the prepared one.
  - Container to load balance: There should be only one option - click `Add to LB`.
  Use HTTPS as a listener and create new Target group (name it for example
  `playground-wt-demo-app-tg`). You can leave `Path pattern` as is, we'll change it
  later. Change health check path to `/`.
  - Service discovery: We're not using that, untick.
1. You should see a running service in your ECS cluster after a few seconds.

#### Travis setup

1. Create a deployment script in `management/deploy-aws.sh` such as:
```sh
#!/bin/bash
ENVIRONMENT=$1

# AWS command opts
TASK_FAMILY="$ENVIRONMENT-wt-demo-app"
SERVICE_NAME="$ENVIRONMENT-wt-demo-app"
AWS_REGION="eu-west-1"

# container setup options
LATEST_TAG=`git describe --abbrev=0 --tags`

# container startup options
WT_CONFIG=$ENVIRONMENT
INFURA_API_KEY=$INFURA_API_KEY # This can be passed from encrypted Travis ENV VARs

# Change port, environment variables, memoryReservation and cpu to be in line with your task
# definition.
TASK_DEF="[{\"portMappings\": [{\"hostPort\": 0,\"protocol\": \"tcp\",\"containerPort\": 8000}],
    \"environment\": [
      {
        \"name\": \"INFURA_API_KEY\",
        \"value\": \"$INFURA_API_KEY\"
      },
      {
        \"name\": \"WT_CONFIG\",
        \"value\": \"$WT_CONFIG\"
      }
    ],
    \"image\": \"xxxxxx.dkr.ecr.region.amazonaws.com/wt-demo-app:$LATEST_TAG-$ENVIRONMENT\",
    \"name\": \"wt-demo-app\",
    \"memoryReservation\": 128,
    \"cpu\": 128
  }]"

echo "Updating task definition"
aws ecs register-task-definition --region $AWS_REGION --family $TASK_FAMILY --container-definitions "$TASK_DEF" > /dev/null
echo "Updating service"
aws ecs update-service --region $AWS_REGION --cluster shared-docker-cluster-t3 --service "$SERVICE_NAME" --task-definition "$TASK_FAMILY" > /dev/null

```
1. Add alias to package.json:
```json
{
  "scripts": {
    "deploy-aws-playground": "./management/deploy-aws.sh playground"
  }
}
```
1. Tell travis to run your deployment script:
```yaml
  - stage: Start service from docker with latest merged tag
    install: true
    sudo: true
    if: branch = release/playground
    script:
    - pip install --user awscli
    - export PATH=$PATH:$HOME/.local/bin
    - npm run deploy-aws-playground
```

And that's it. This will on every push to `release/playground` look up the last
tag in this branch, look for a docker image with such tag and start a service
from that image.

### Networking

This will allow traffic from internet to be routed by the load balancer and 
served by your container.

#### Preconditions

- Running Load Balancer.
- DNS Access (Cloudflare).

#### AWS setup

1. Setup routing on ELB's HTTPS route (EC2 -> Load Balancers -> Listeners tab ->
View/edit rules). You will add a new rule where if `Host == demo.windingtree.com`,
the traffic should be Forwarded to a target group you have created in the previous
step. If there's a routing rule added by previous step for the same target group,
delete it.
1. Create a certificate in Certificate manager for `demo.windingtree.com`. Pick
DNS verification. It will require you to create a new `CNAME` DNS record on Cloudflare
to verify domain ownership. It should take no more than a few minutes.
1. Back in Load balancer Listeners, add your fresh certificate to HTTPS listener.
1. In DNS set another CNAME record for `demo.windingtree.com` that points to the
load balancer DNS name.
1. That's it! Your service should now be responding on `demo.windingtree.com`.