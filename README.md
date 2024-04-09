# AWS Account Transfer Project

## 1. Overview 

This project will reduce Chatloop’s AWS costs by transferring its AWS infrastructure to a new AWS account that has not-for-profit benefits.

## 2. Context

### 2. 1 Organisations

There are two organisations involved in Chatloop, each with their own responsibilities.

- **Neon King Kong Pty Ltd** is a software development social enterprise. They build and maintain the Chatloop apps, and grant a free, perpetual licence to Chatloop Ltd to use the apps.
- **Chatloop Ltd** is a registered charity that manages the Chatloop program in Australia. They are responsible for a wide range of things, such as recruiting and training volunteers, building partnerships with other organisations, applying for grant funding etc.

Currently, Chatloop is being hosted on Neon King Kong’s AWS account, which is costing Neon King Kong approximately AUD$3500 each year.

Chatloop Ltd will be paying for this account transfer project.

### 2.2 Apps

#### `chatloop-monolith`

A RESTful and web sockets server that provides most of Chatloop’s backend

#### `chatloop-training`

A graphql server that acts as a micro service for the in-app volunteer training programs.

#### `chatloop-admin`

A React web app where admin users can oversee the whole Chatloop program.

#### `chatloop-refer`

A React web app where referrers can refer their clients to Chatloop

#### `chatloop-website`

A mostly static, public-facing website for Chatloop Ltd.

#### `chatloop-mobile`

A React Native mobile app that is used by refugees learning English, volunteer conversation partners, and volunteer tutors.

With the exception of `chatloop-mobile`, all of these apps are hosted on AWS and are included in this project’s scope.

### 2.3 AWS services

#### 2.3.1 RDS

We use three RDS databases (PostgreSQL), for

- `chatloop-monolith` production
- `chatloop-monolith` staging. Also used for CI/CD E2E tests
- `chatloop-training` production

#### 2.3.2 Elastic Beanstalk (EB)

We have three EB environments running:

- `chatloop-monolith` production
- `chatloop-monolith` staging. Also used for CI/CD E2E tests
- `chatloop-training` production

Each environment use a range of AWS services, including EC2, Virtual Private Cloud etc

#### 2.3.3 Elastic Load Balancing (ELB)

Although AWS sets up a separate load balancer for each EB environment, we have been able to save money be configuring each EB environment to use the same load balancer. In other words, we only have one load balancer, used by all three EB environments.

#### 2.3.4 Route 53

We use one hosted zone for Chatloop, with domain registration for `chatloop.org`

#### 2.3.5 WAF

I set this up to try to filter out endpoint spamming, but it doesn’t work.

#### 2.3.6 Secrets Manager

We have 7 secrets in the Secret Manager, but I’m not sure they’re being used properly.

#### 2.3.7 S3

We have quite a few S3 buckets, but many of them aren’t being used, or aren’t being used properly.
- `avatar-chatloop`  - for storing user profile photos
- `avatar-chatloop-dev`  - development version of `avatar-chatloop`
- `chatloop-admin` - hosts the `chatloop-admin` web app
- `dev-chatloop-admin` 
- `chatloop-refer` - hosts the `chatloop-refer` web app
- `chatloop-refer-staging`
- `chatloop.org` - hosts `chatloop-website`. This may be old: need to check
- `chatloop-public-resources` - various resources (videos, images, pdfs etc) used by other Chatloop apps
- `www.chatloop.org` - more public resources, should be combined with `chatloop-public-resources` 
- `chatloop-terraform` - used by the `chatloop-monolith` Terraform deployment
- `chatloop-react` - old PWA
- `chatloop.io` - old static website
- `www.chatloop.io` - empty
- `athena-production-api-nlb-logs`
- `production-api-nlb-logs`
- `production-api-v2-nlb-logs`
- `shared-alb-logs-eb`
- `staging-api-nlb-logs`
- `staging-api-v2-nlb-logs`

#### 2.3.8 CloudFront

Our S3 bucket web apps are distributed via CloudFront

### 2.4 Bitbucket

Bitbucket hosts all our code repositories, and we use Bitbucket Pipelines for CI/CD.

`chatloop-monolith` runs JUnit unit and integration tests in the pipeline, and deploys as a Docker container to Elastic Beanstalk (production and staging)

`chatloop-training` runs JUnit unit and integration tests in the pipeline, and deploys as a Docker container to Elastic Beanstalk (production and staging)

`chatloop-refer` runs Jest unit tests and Cypress E2E tests on working branches. Deploys to S3 (staging and production) on the `main` branch.

`chatloop-admin` runs Cypress E2E test and deploys to S3 (staging and production)

`chatloop-monolith-au` (misnamed, it should be called `chatloop-website-au`) does NOT use Pipelines for deployment. It is deployed via Vercel.

### 2.5 Terraform

Terraform has been set up for `chatloop-monolith`, but it never gets used. It might be something we want to consider for this project. Documentation is in `chatloop-monolith`’s readme, and the Terraform code probably needs to be updated.

### 2.6 Existing AWS cost-reduction practices

We are already doing several things to keep our AWS costs down, and we’ll continue to do these after the transfer project is complete.

- Reserves instances (EC2, RDS)
- Single Elastic Load Balancer
- The staging environment for `chatloop-training` is turned off.

## 3. Outcomes

By the end of the project, all of the Chatloop backend will be transferred to the new AWS account and removed from the old AWS account.

All Chatloop apps will continue to function exactly as they did prior to the transfer: same endpoint URLs, same environment variables, same CI/CD process etc.

The monthly AWS costs will be reduced.

Also, the following outcomes are optional “stretch goals”:

- better logging
- better management of secrets and environment variables
- prevention of endpoint spamming
- on-demand E2E testing and staging environments
- groundwork laid for future multi-region deployment that can comply with data sovereignty regulations.
- private access to avatar S3 buckets

## 4. Timelines

Because the AWS Nonprofit Credit program grants credit that is valid for a financial year (1 July - 30 June), the optimal time to transfer to the new account is 1 July 2024.

Work on the project will begin on 1 April 2024.

That gives us 3 months to prepare for the account transfer.

So the rough project timeline is:
| Dates           | Task                                 |
|-----------------|--------------------------------------|
| 1 Apr - 30 Apr  | Planning                             |
| 1 May - 30 Jun  | Set up new AWS account               |
| 1 Jul           | Transfer to new account              |
| 1 Jul - 7 Jul   | Remove Chatloop from old AWS account |
| 1 Jul - 31 July | Monitor for problems                 |


## 5. New ways to reduce costs

- Use AWS free tier for first 12 months, use AWS Nonprofit credit for subsequent years
- Consolidate databases
- Single load balancer 
- Don’t have staging/e2e environment running all the time. Dockerised `chatloop-monolith`, `chatloop-training`, and db combination so that it’s easy to fire up e2e/staging environment backend services
