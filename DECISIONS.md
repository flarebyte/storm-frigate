# Architecture decision records

An [architecture
decision](https://cloud.google.com/architecture/architecture-decision-records)
is a software design choice that evaluates:

- a functional requirement (features).
- a non-functional requirement (technologies, methodologies, libraries).

The purpose is to understand the reasons behind the current architecture, so
they can be carried-on or re-visited in the future.

## Initial idea

Notes:

Integration source (private) produces a deployment-candidate package with:

- source code: monorepo
- resource code: monorepo: ex: script that produce cloudformation based on configuration.
- distribution code: monorepo: ex: lamba A code, step function B, static files C
- metadata and reports: (unit test cov, version, ...)
- runbook and dashboard generation script

Copy this deployment package to temp S3 bucket (7 days retention, write once at key)

CodeBuild run on deployment candidate package, verifies it, and add extra metadata and stats (scc, security check, linting ...)
Ask for manual approval and sign package ?
Create an official deployment package on long term S3 storage (write once)
but also:

- add runbook, stats, metadata in an artifacts bucket (possibly to db but may be overkill)

Now that the deployment-package is ready, it can be run with a configuration to create a deployment flavour.
`sf deployWith deploymentId configurationId` to get a service Id.

Configuration should have key-value style tag that allows us to define something like: app:applicationName, version: 123, commitId:commitId, branch:main, flavour:featureA, audience:qa

Lambda code and State Machine Definition in S3.

Resources sync management: AWS resources, files, static web scripts to update these on demand.

Deployment is Resources script + configuration + S3 refs (lambda, step functions files).

