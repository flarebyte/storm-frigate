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

Integration source (private) produces a deploymen-candidate package with:

- source code
- distribution
- metadata: (unit test cov, version, ...)
- config key values or config id: kv better because versioned and cannot be lost

Copy this deployment package to temp S3 bucket (7 days retention, write once at key)

CodeBuild run on deployment-candidate package, verifies it, and add extra metadata and stats (scc, ...)
Possibly hydrate cdk but do sync ?
Possibly run some candidate scripts for hydrating stuff
Add cloudformation to package and order of run.
Ask for manual approval and sign package ?
Possibly create a deployment script.
Generate runbook and dashboard and add to package.
Create an official deployment package on long term S3 storage (write once)
but also:

- add runbook, stats, metadata in an artifacts bucket (possibly db but may be overkill)

Now that the deployment-package is ready, it can be deployed
`sf deploy id`
It should be possible to: `sf service list`
