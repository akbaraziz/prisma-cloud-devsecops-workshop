# Welcome!

This workshop will demonstrate how to leverage infrastructure as code (IaC) and DevSecOps patterns to automate, scale, and improve the security posture of cloud infrastructure and applications. We will create a pipeline that ensures our configurations are secure and compliant from code to cloud.

This guide provides step-by-step instructions to integrate **Prisma Cloud** (and **checkov**) with **Terraform Cloud, Github, VScode** and **AWS**. 


![Diagram of workshop flow](images/0_flow_diagram.png "Diagram of workshop flow")

## Learning Objectives
- Gain an understanding of DevSecOps and infrastructure as code (IaC) using Terraform
- Scan IaC files for misconfigurations locally
- Set up CI/CD pipelines to automate security scanning and policy enforcement
- Fix security findings and AWS resource misconfigurations with Prisma Cloud

**Let’s start with a few core concepts...**

### DevSecOps
The foundation of DevSecOps lies in the DevOps movement, wherein development and operations functions have merged to make deployments faster, safer, and more repeatable. Common DevOps practices include automated infrastructure build pipelines (CI/CD) and version-controlled manifests (GitOps) to make it easier to control cloud deployments. By baking software and infrastructure quality requirements into the release lifecycle, teams save time manually reviewing code, letting teams focus on shipping features.

As deployments to production speed up, however, many traditional cloud security concepts break down. With the rise of containerized technologies, serverless functions, and IaC frameworks, it is increasingly harder to maintain visibility of cloud security posture. 

By leveraging DevOps foundations, security and development teams can build security scanning and policy enforcement into automated pipelines. The ultimate goal with DevSecOps is to “shift cloud security left.” That means automating it and embedding it earlier into the development lifecycle so that actions can be taken earlier. Preventing risky deployments is a more proactive approach to traditional cloud security that often slows down development teams with deployment rollbacks and disruptive fixes.

For DevSecOps to be successful for teams working to build and secure infrastructure, embracing existing tools and workflows is critical. At Palo Alto Networks, we’re committed to making it as simple, effective, and painless as possible to automate security controls and integrate them seamlessly into standard workflows.




### Infrastructure as Code Using Terraform
Infrastructure as code (IaC) frameworks, such as  HashiCorp Terraform, make cloud provisioning scalable and straightforward by leveraging automation and code. Defining our cloud infrastructure in code simplifies repetitive DevOps tasks and gives us a versioned, auditable source of truth for the state of an environment.

Terraform is useful for defining resource configurations and interacting with APIs in codified, stateful manor. Any updates we want to make, such as adding more instances, or changes to a configuration, can be handled by Terraform. 

For example, the following Terraform resource block defines a simple AWS S3 bucket:

```hcl
resource "aws_s3_bucket" "data" {
  bucket     = "my_bucket_name"
  acl        = "public-read-write"
}
```

After performing `terraform init`, we can provision an S3 bucket with the following command:

```bash
terraform apply
```

Any changes made to the resource definition within a .tf file, such as adding tags or changing the acl, can be pushed with the  `terraform apply` command. 

Another benefit of using Terraform to define infrastructure is that code can be scanned for  misconfigurations before the resource is created. This allows for security controls to be integrated into the development process, preventing issues from ever being introduced, deployed and exploited.


## Setup / Prerequisities
- Github account
- Terraform Cloud account
- AWS account (provided during workshop)
- Prisma Cloud account (OPTIONAL)

## Log into AWS Event Engine
> [!NOTE]
> *This section is for live workshop events only.*

Follow the link provided to you for Event Engine. Enter the event hash (if it is not already populated), and click `Accept Terms & Login`.

![ee-login](images/ee-login.png)

Choose your perferred sign-in method (OTP is recommended for workshop attendees).

![ee-sign-in](images/ee-sign-in.png)

Once logged in, copy the provided credentials into a local text file and then click `Open Console` to navigate to AWS.

![ee-creds](images/ee-creds.png)

## Configure IAM User and API Key
*Skip this section for workshops <100 attendees*

From the AWS console, select `IAM` or search for 'IAM' in the Search bar if not displayed.

![iam-console-home](images/iam-console-home.png)

In the IAM Dashboard, click `Users` on the left sidebar.

![iam-dashboard](images/iam-dashboard.png)

Click the `Create User` button on the top right.

![create-user](images/iam-create-user.png)

Specify a `User name` that will be unique then click `Next`.

![user-details](images/iam-user-details.png)

Next, set the permissions for the user by selecting `Attach policies directly` and attaching the `AdministratorAccess` policy.

![iam-set-perms](images/iam-set-perms.png)

Review the user details and click `Create user`.

![iam-review-create](images/iam-review-create.png)

Now we need to assign an API key to the user we just created. Click on the user you just created from the IAM Dashboard and then click `Create access key` on the right.

![iam-create-key](images/iam-create-key.png)

Select `Other` from the Access Key options.

![iam-key-options](images/iam-key-options.png)

Optionally, supply a tag to associate with the Access Key, then click `Create access key`.

![iam-key-tag-and-create](images/iam-key-tag-and-create.png)

Finally, save the Access Key data provided (copy to a local file). This credential will be used to deploy resources to AWS in a later section. When ready, click `Done`.

![iam-retrieve-key-data](images/iam-retrieve-key-data.png)

An access key will now appear on the User details page.

![iam-key-created](images/iam-key-created.png)


## Configure Cloud9
To ensure we all have the same environent configuration, we will use Cloud9, a cloud-delivered IDE from AWS, to carry out many of the steps in this workshop. To set this up, navigate back to the AWS Console home and select Cloud9 (or enter it into the Search bar).


![c9-console-home](images/c9-console-home.png)

On the Cloud9 Environments page, click `Create Environment`

![c9-create-env](images/c9-create-env.png)

Enter a `Name` for the Environemnt and select `New EC2 instance` for Environment Type.

![c9-env-options1](images/c9-env-options1.png)

Select `Additional instance types` then choose `t3.medium` from the drop-down.

![c9-env-options2](images/c9-env-options2.png)

Leave all other options on default setting and click `Create`. 

![c9-env-options3](images/c9-env-options3.png)


Once the environment is created, navigate to it and click `Open in Cloud9` to launch the IDE.

![c9-open-ide](images/c9-open-ide.png)

Close all of the Welcome default windows, then create a new Terminal window.

![c9-close-welcome](images/c9-close-welcome.png)

![c9-open-term](images/c9-open-term.png)

![c9-blank-term](images/c9-blank-shell.png)

Congrats! Cloud9 is now ready to use. Before installing checkov or pulling code to scan, create and activate a python virtual environment to better organize python packages.

``` 
python3 -m venv env
source ./env/bin/activate 
```

![c9-py-venv](images/c9-py-venv.png)

##
# Section 1: Code Scanning with checkov

[Checkov](https://checkov.io) is an open source 'policy-as-code' tool that scans cloud infrastructure defintions to find misconfigurations before they are deployed. Some of the key benefits of checkov:
1. runs as a command line interface (CLI) tool 
2. supports many common plaftorms and frameworks 
3. ships with thousands of default policies
4. works on windows/mac/linux (any system with python installed)

## Install checkov

To get started, install checkov using pip:

```
pip3 install checkov
```

![](images/c9-install-checkov.png)




Use the `--version` and `--help` flags to verify the install and view usage / optional arguements 

```
checkov --version
checkov --help
```
![](images/c9-checkov-options.png)

To see a list of every policy that Checkov can enforce, use the `-l` or ` --list` options.

```
checkov --list
```

Now that you see what checkov can do, let's get some code to scan...



## Fork and clone target repository
This workshop involves code that is vulnerable-by-design. All of the necessary code is contained within [this repository](https://github.com/paloAltoNetworks/prisma-cloud-devsecops-workshop) or workshop guide itself.

To begin, log into Github and navigate to the [Prisma Cloud DevSecOps Workshop](https://github.com/paloAltoNetworks/prisma-cloud-devsecops-workshop) repository. Create a `Fork` of this repositry to create a copy of the code in your own account.

![](images/gh-fork.png)

Ensure the selected `Owner` matches your username, then proceed to fork the repository by clicking `Create fork`.

![](images/gh-create-fork.png)

Grab the repo URL from Github, then clone the **forked** repository to Cloud9.

![](images/gh-clone.png)

```
git clone https://github.com/<your-organization>/prisma-cloud-devsecops-workshop.git
cd prisma-cloud-devsecops-workshop/
git status

```

![](images/c9-git-clone.png)


Great! Now we have some code to scan. Let's jump in...



## Scan with checkov

Checkov can be configured to scan files and enforce policies in many different ways. To highlight a few: 
1. Scans can run on individual files or entire directories. 
2. Policies can be selected through selection or omission. 
3. Enforcement can be determined by flags that control checkov's exit code.


Let's start by scanning the entire `./code` directory and viewing the results.

```
cd code/
checkov -d .
```
![](images/c9-checkov-d.png)

Failed checks are returned containing: the offending file and resource, the lines of code that triggered the policy, and a guide to fix the issue.

![](images/checkov-result.png)

Now try running checkov on an individual file with `checkov -f <filename>`. 

```
checkov -f deployment_ec2.tf
```
```
checkov -f simple_ec2.tf
```

**Question:** Why are there more security findings for `deployment_ec2.tf` than there are for `simple_ec2.tf`? What about `simple_s3.tf` vs `simple_ec2.tf`?


Policies can be optionally enforced or skipped with the `--check` and `--skip-check` flags. 

```
checkov -f deployment_s3.tf --check CKV_AWS_18,CKV_AWS_52
```
```
checkov -f deployment_s3.tf --skip-check CKV_AWS_18,CKV_AWS_52
```

Frameworks can also be selected or omitted for a particular scan:


```
checkov -d . --framework secrets
```
```
checkov -d . --skip-framework dockerfile
```

![](images/checkov-secrets.png)


Lastly, enforcement can be more granularly controlled by using the `--soft-fail` option. Applying `--soft-fail` results in the scan always returning a 0 exit code. Using `--hard-fail-on` overrides this option. 

Check the exit code when running `checkov -d . ` with and without the `--soft-fail` option.

```
checkov -d . ; echo $?
```
```
checkov -d . --soft-fail ; echo $?
```

An example of using `--soft-fail` and exit codes in a pipeline context will be demosntrated in a later section.


## Custom Policies

Checkov supports the creation of [Custom Policies](https://www.checkov.io/3.Custom%20Policies/YAML%20Custom%20Policies.html) for users to customize their own policy and configuration checks. Custom policies can be written in YAML (recommended) or python and applied with the `--external-checks-dir` or `--external-checks-git` flags. 

Let's create a custom policy to check for [local-exec and remote-exec Provisioners](https://developer.hashicorp.com/terraform/language/resources/provisioners/local-exec) being used in Terraform resource definitons. (Follow link to learn more about provisioners and why it is a good idea to check for them).

```yaml
metadata:
 name: "Terraform contains local-exec and/or remote-exec provisioner"
 id: "CKV2_TF_1"
 category: "GENERAL_SECURITY"
definition:
 and:
  - cond_type: "attribute"
    resource_types: all 
    attribute: "provisioner/local-exec"
    operator: "not_exists"
  - cond_type: "attribute"
    resource_types: all
    attribute: "provisioner/remote-exec"
    operator: "not_exists"
```
Add the above code to a new file within a new direcotry.

```
mkdir custom-checks/
vim custom-checks/check.yaml
```
>[!TIP]
> use `echo '$(file_contents)' > custom-checks/check.yaml` if formatting is an issue with vim.


Save the file. Then run checkov with the `--external-checks-dir` to test the custom policy.

```
checkov -f simple_ec2.tf --external-checks-dir custom-checks
```
![](images/checkov-custom-checks.png)

**Challenge:** write a custom policy to check all resources for the presence tags. Specifically, ensure that a tag named "Environment" exists.


## IDE plugin
> [!NOTE]
> *Demo Only. Requires API key for Prisma Cloud.*

[Prisma Cloud IDE plugins](https://docs.prismacloud.io/en/classic/appsec-admin-guide/get-started/connect-your-repositories/integrate-ide/integrate-ide)

[VScode extension](https://marketplace.visualstudio.com/items?itemName=Bridgecrew.checkov)


## Integrate with Github Actions
- checkov
- yor
- results in GHAS
- terraform / TFC flow*
- BONUS: link to pre-commit hooks

## Integrate with Terraform Cloud
...

## Block a Pull Request, Prevent a Deployment
...

##
# Section 2: Application Security with Prisma Cloud
> [!NOTE]
> *This portion of the workshop is inteded to be view-only. Those with existing access to Prisma Cloud can follow along but is not recommended to onboard any of the workshop content into a production deployment of Prisma Cloud. Use this guide as an example and the content within for demonstration purposes only.*

## Integrations and Dashboards
- Checkov/GHA with API key (GHAS, severity, image scanning)
- TFC Run Tasks (streamlined output)
- Github Application (PR comments)
- Results in platform (for each)
    - Projects, IaC, Vulns/SCA, SBOM, CICD, etc.

## Issue a PR Fix
...

## Drift Detection
...
- seperate example

# Conclusion
- congrats! and review of material
- some call to action and links ... 












