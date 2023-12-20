# Welcome!

In this workshop, we’ll learn how to leverage infrastructure as code (IaC) and DevSecOps patterns to automate, scale, and improve the security posture of cloud infrastructure and applications. We’ll create a pipeline that ensures our configurations are secure and compliant from build-time to runtime.

This guide will demonstrate how to secure applications from code to cloud by integrating **Prisma Cloud** (and **checkov**) with **Terraform Cloud, Github, VScode** and **AWS**. 

![Diagram of workshop flow](images/0_flow_diagram.png "Diagram of workshop flow")

## Learning Objectives
- Gain an understanding of DevSecOps and infrastructure as code (IaC) using Terraform
- Scan IaC files for misconfigurations locally
- Set up CI/CD pipelines to automate security scanning and policy enforcement
- Fix security findings and AWS resource misconfigurations with Prisma Cloud

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
- --version, --help, --list

##
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
- VScode extension
- CLI + API Key (severity, policy name)

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
*This portion of the workshop is optional (or 'read-only') unless you have existing access to Prisma Cloud.*

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
...












