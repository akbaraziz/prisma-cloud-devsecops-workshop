# Welcome!

In this workshop, you’ll learn how to leverage infrastructure as code (IaC) and DevSecOps patterns to automate, scale, and improve the security posture of your cloud infrastructure. We’ll create a pipeline that provides frequent, easy-to-digest improvements to ensure our configurations are secure and compliant from the build-time to runtime.

Using **Prisma Cloud, Checkov, VS Code, GitHub, Terraform Cloud,** and **AWS**, we’ll get hands-on experience implementing an automated **Terraform** security and compliance workflow.

![Diagram of workshop flow](images/0_flow_diagram.png "Diagram of workshop flow")

## Learning Objectives
- Get an overview of DevSecOps and Terraform infrastructure as code (IaC)
- Scan IaC files for misconfigurations locally
- Set up CI/CD pipelines to automate security scanning and policy enforcement
- Fix IaC security errors and AWS resource misconfigurations with Prisma Cloud

**Let’s start with a few core concepts!**

### IaC Using Terraform
...
### DevSecOps
...

## Setup / Prerequisities
- Github account
- Terraform Cloud account
- AWS account (provided during workshop)
- Prisma Cloud account (OPTIONAL)

## Log into AWS Event Engine
...

## Configure Cloud9
...

## Install checkov
...
- --version, --list

# Section 1: Code Scanning with Open Source Tools

## Fork and clone target repository
...

## Scan with checkov
- -f, -d
- -skip, -check
- -framework
- -soft, -hard, exit codes and automation

## Custom Policies
- -external-checks-dir

## IDE plugin
*Demo Only. Requires API key for Prisma Cloud.*

## Integrate with Github Actions
- checkov
- yor
- results in GHAS
- terraform / TFC flow*
- BONUS: pre-commit hooks

## Integrate with Terraform Cloud
...

## Block a Pull Request, Prevent a Deployment
...

# Section 2: Application Security with Prisma Cloud
*This portion of the workshop is optional (or 'read-only') unless you have existing access to Prisma Cloud.*

## Integrations and Dashboards
- Checkov/GHA with API key (GHAS, severity)
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
...












