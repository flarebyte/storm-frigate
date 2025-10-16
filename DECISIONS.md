# Architecture decision records

An [architecture
decision](https://cloud.google.com/architecture/architecture-decision-records)
is a software design choice that evaluates:

- a functional requirement (features).
- a non-functional requirement (technologies, methodologies, libraries).

The purpose is to understand the reasons behind the current architecture, so
they can be carried-on or re-visited in the future.

## Initial idea

**Problem Overview**
Enable continuous deployment orchestration in AWS for multiple services that share implementation artifacts across deployment environments (e.g., `int`, `live`, `qa`, `featureA`). The system must handle packaging, verification, manual approval, and deployment execution based on configuration.

---

**Deployment Package Contents**
All content is stored in a monorepo and bundled per deployment candidate:

- Source code
- Resource scripts (e.g., generate CloudFormation)
- Distribution assets (e.g., Lambda A, Step Function B, Static Assets C)
- Metadata (e.g., unit test coverage, version info)
- Runbook and dashboard generation scripts

---

**Flow and Tooling Requirements**

1. **Candidate Package Creation**

   - Produced by an integration source (private)
   - Copied to a temporary S3 bucket (write-once, 7-day retention)
   - Must include all expected deployment components and metadata

2. **CodeBuild Validation Phase**

   - Runs on the candidate package
   - Performs:

     - Package verification
     - Code stats (SCC)
     - Security checks
     - Linting

   - Augments metadata and stats

3. **Manual Approval**

   - Optional step for human approval
   - Supports digital signing or tagging of approved packages

4. **Promotion to Official Deployment Package**

   - Stored in long-term S3 (immutable/write-once)
   - Includes runbook, metadata, stats
   - Artifacts also saved in a secondary bucket (for analysis or indexing)
   - Optionally indexed into a database (not required initially)

---

**Deployment Execution**

- A deployment is created using:

  - The official deployment package
  - A configuration (defines environment-specific details)

- Configuration tags (key-value):

  - `app`, `version`, `commitId`, `branch`, `flavour`, `audience`

- Executed via CLI or API:

  - `sf deployWith deploymentId configurationId`
  - Returns a unique service deployment ID

- Deployment includes:

  - Resource scripts
  - Configuration (tags and values)
  - References to S3-stored Lambda and State Machine definitions

---

**Use Cases**

- Deploy same implementation to `qa`, `int`, `live` with different configurations
- Use shared scripts to generate unique CloudFormation stacks per environment
- Reference pre-uploaded Lambda zip files and State Machine definitions from S3
- Reuse verified deployment package for different flavours (e.g., `featureA`, `featureB`)
- Manually approve and sign off before production deployment
- Store long-term audit artifacts separately for compliance or analysis

---

**Edge Cases**

- Missing or malformed metadata in candidate package
- Failure during CodeBuild verification or metadata enrichment
- Rejection during manual approval
- Attempt to re-upload to write-once S3 paths
- Conflicts between tag keys or duplicated configurations
- Configuration referencing missing S3 resources
- Execution in a non-existent or deprecated flavour

---

**Limitations**

- Does not define how the CloudFormation scripts are written or structured
- Does not handle deployment rollback or state management
- Does not support dynamic package editing post-verification
- No real-time monitoring or alerting built into the flow
- Database indexing is optional and should not block deployment
