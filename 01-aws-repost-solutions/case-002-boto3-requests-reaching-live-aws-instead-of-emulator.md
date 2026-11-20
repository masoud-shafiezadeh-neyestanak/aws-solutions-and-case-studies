# Case 002 — Boto3 Requests Reaching Live AWS Instead of AWS Emulator

> An independent research note based on an AWS re:Post discussion. The scenario and findings are summarized in original wording; the original question and shared answer are not reproduced.

## 1. Source Discussion

- **Platform:** AWS re:Post
- **Discussion title:** AWS Emulator connected to my AWS account!
- **Discussion URL:** [AWS Emulator connected to my AWS account!](https://repost.aws/questions/QUE3hHpAqESUmk-o0XKEye6g/aws-emulator-connected-to-my-aws-account#ANc6d8NmK9RxysTPUl7TpEDQ)
- **Date reviewed:** 2026-11-20
- **My participation:** Answer published in the discussion
- **Thread outcome:** Answer accepted by the question author and marked Expert Approved on re:Post

The Accepted and Expert Approved labels describe outcomes on the re:Post platform. They are not an official AWS certification or a guarantee that every part of the analysis has been independently reproduced.

## 2. Technical Scenario

A user ran AWS Emulator locally in a Windows Docker environment and used Boto3 to list Amazon S3 buckets. The returned bucket list appeared to contain resources from the user's real AWS account, raising the concern that the emulator might have been initialized with a copy of the account's data.

The key diagnostic question is whether the Boto3 client was actually sending its API requests to the local emulator endpoint or to the normal AWS service endpoint.

## 3. AWS Services and Context

- **Local tool:** AWS Emulator
- **Runtime:** Docker on Windows
- **SDK:** Boto3 (AWS SDK for Python)
- **AWS service being called:** Amazon S3
- **Diagnostic service:** AWS Security Token Service (AWS STS)
- **Example operation:** `ListBuckets`

The source discussion included account-specific bucket names and ARNs. Those values are intentionally omitted from this note.

## 4. Key Technical Questions

1. Does AWS Emulator import or synchronize resources from a user's real AWS account?
2. What determines whether a Boto3 client sends a request to AWS or to the local emulator?
3. How can a developer confirm which endpoint and account context a test process is using?
4. Which environment variables and local configuration files are required for AWS Emulator?
5. How can accidental use of real AWS credentials be avoided during local testing?

## 5. Investigation and Evidence

The current AWS Emulator documentation establishes the following:

| Finding | Evidence | Confidence |
|---|---|---|
| AWS Emulator is a local implementation of AWS APIs; requests are sent to the emulator endpoint rather than to AWS when correctly configured. | AWS Emulator User Guide — What is AWS Emulator? | Documented |
| The emulator keeps its simulated resources in memory and deletes them when it stops. It does not import the user's live AWS resource inventory. | AWS Emulator User Guide — What is AWS Emulator? and Security in AWS Emulator | Documented |
| Each emulator uses the account ID `000000000000`. | AWS Emulator command reference — Credentials and authentication | Documented |
| The SDK must be explicitly configured to send requests to the emulator. AWS documents the global `AWS_ENDPOINT_URL` environment variable and the generated local SDK configuration. | AWS Emulator User Guide — Pointing your AWS SDK at AWS Emulator | Documented |
| If `AWS_ENDPOINT_URL` is not set in the shell that runs the tests, the code calls AWS using whatever credentials it can find. | AWS Emulator User Guide — Pointing your AWS SDK at AWS Emulator | Documented |
| The emulator generates its own credentials for each run and, by default, rejects requests signed with other credentials. | AWS Emulator User Guide — Credentials and authentication | Documented |
| Boto3 can use the emulator's certificate through `AWS_CA_BUNDLE`. | AWS Emulator User Guide — Pointing your AWS SDK at AWS Emulator | Documented |

## 6. Analysis

The reported S3 response strongly suggests that the request reached the real AWS account rather than the local emulator.

The Boto3 code in the discussion created a client without passing an explicit `endpoint_url`:

```python
import boto3

s3 = boto3.client("s3")
print(s3.list_buckets()["Buckets"])
```

This code is not inherently tied to either AWS or the emulator. Its destination depends on the SDK configuration available to the Python process.

For example, Boto3 can use a custom endpoint configured through:

- The global `AWS_ENDPOINT_URL` environment variable.
- A service-specific endpoint variable such as `AWS_ENDPOINT_URL_S3`.
- An `endpoint_url` setting in the selected AWS shared config profile.
- An explicit `endpoint_url` passed when creating the client.

If none of these settings directs the client to the emulator, Boto3 uses the normal AWS endpoint resolution and the credentials available through its credential provider chain.

Consequently, the emulator does not need to have copied the user's buckets. A more plausible explanation is that the Python process was not using the emulator endpoint and therefore queried live Amazon S3.

This is a strong diagnosis based on the reported output, but the exact environment-variable and profile state from the original machine was not supplied. The endpoint should be verified directly rather than inferred from the bucket list alone.

## 7. Proposed Troubleshooting Approach

### Step 1 — Configure the emulator in the same shell that runs Python

For a Windows PowerShell session using the default emulator port, the AWS documentation shows the following configuration pattern:

```powershell
$env:AWS_ENDPOINT_URL = "https://127.0.0.1:9297"
$env:AWS_SHARED_CREDENTIALS_FILE = "$env:USERPROFILE\.aws\emulator\9297\emulator_credentials"
$env:AWS_CONFIG_FILE = "$env:USERPROFILE\.aws\emulator\9297\config"
$env:AWS_CA_BUNDLE = "$env:USERPROFILE\.aws\emulator\9297\ca.pem"
$env:AWS_EC2_METADATA_DISABLED = "true"
$env:AWS_DISABLE_HOST_PREFIX_INJECTION = "true"
$env:AWS_REGION = "us-east-1"
$env:AWS_DEFAULT_REGION = "us-east-1"
```

Use the actual port and file paths generated by the emulator if they differ from these examples. When the emulator runs in Docker, follow its Docker setup instructions so that the host can reach the published local endpoint.

### Step 2 — Remove environment variables that can provide real AWS credentials

Before running tests, remove any real credential variables from that PowerShell session:

```powershell
Remove-Item Env:AWS_ACCESS_KEY_ID, Env:AWS_SECRET_ACCESS_KEY, Env:AWS_SESSION_TOKEN, Env:AWS_PROFILE -ErrorAction SilentlyContinue
```

This is important because environment credentials take precedence over credentials in the shared credentials file. The emulator generates a separate credential pair for its own run.

Do not put real AWS credentials into the emulator configuration or test environment.

### Step 3 — Verify the caller identity before making service calls

Run this command from the same shell:

```powershell
aws sts get-caller-identity
```

Or verify it from the same Python process that will run the test:

```python
import boto3

sts = boto3.client("sts")
print(sts.get_caller_identity())
```

Expected interpretation:

| Returned account ID | Meaning |
|---|---|
| `000000000000` | The request reached AWS Emulator |
| The user's actual AWS account ID | The request reached the real AWS environment |

If the real account ID appears, stop the test and correct the endpoint and credential configuration before making any further calls.

### Step 4 — Run the S3 test again

Once the STS identity confirms the emulator account, run the original S3 list operation again. The response should reflect resources in the emulator's local in-memory state, not the live account's bucket inventory.

### Step 5 — Keep the test environment isolated

Run the emulator privately, keep authentication enabled, and do not expose its port to the internet. AWS also recommends keeping real credentials and production data out of emulator tests.

## 8. Validation Status

- [x] Relevant AWS Emulator documentation reviewed
- [x] AWS SDK endpoint configuration documentation reviewed
- [ ] Original machine configuration independently inspected
- [ ] Original behavior independently reproduced
- [ ] Corrected configuration independently tested against the reported environment
- [x] Answer posted in the linked re:Post discussion
- [x] Answer accepted by the question author
- [x] Answer marked Expert Approved on re:Post

**Root-cause assessment:** The most likely explanation is that the Boto3 process used the normal AWS endpoint because the emulator endpoint configuration was missing, not inherited by the Python process, or overridden by another endpoint setting. The exact cause on the original machine remains unverified.

## 9. What I Learned

- An AWS SDK client does not automatically know that a local emulator is running. Its endpoint must be configured.
- A call such as `boto3.client("s3")` can target either AWS or an emulator, depending on the active SDK configuration.
- AWS STS `GetCallerIdentity` is a simple and useful diagnostic for checking which account context receives a request.
- Endpoint selection and credential selection are related but separate configuration concerns. Pointing to a local endpoint while retaining real credentials is not a safe testing setup.
- Environment variables must be set in the environment of the process that runs the tests.
- A successful API response is not proof that a request reached the intended environment.

## 10. References

1. [AWS Emulator — Pointing your AWS SDK at AWS Emulator](https://docs.aws.amazon.com/emulator/latest/userguide/point-sdk-to-emulator.html) — Endpoint setup, Windows PowerShell variables, credentials, certificate configuration, and STS verification.
2. [AWS Emulator — What is AWS Emulator?](https://docs.aws.amazon.com/emulator/latest/userguide/what-is-service.html) — Local operation, simulated resources, and emulator limitations.
3. [AWS Emulator — Command reference](https://docs.aws.amazon.com/emulator/latest/userguide/cli-reference.html) — Default endpoint, local files, generated credentials, and the emulator account ID.
4. [AWS Emulator — Security](https://docs.aws.amazon.com/emulator/latest/userguide/security.html) — Keeping the emulator private and excluding real credentials and data.
5. [AWS SDKs and Tools — Service-specific endpoints](https://docs.aws.amazon.com/sdkref/latest/guide/feature-ss-endpoints.html) — Endpoint configuration and precedence rules.

## 11. Privacy and Attribution Review

- [x] Scenario summarized in original wording
- [x] Other users' full question and answer text not reproduced
- [x] Original bucket names and account-specific ARNs omitted
- [x] AWS documentation linked
- [x] Accepted and Expert Approved statuses attributed to re:Post
- [x] No unsupported claim that the emulator copied live AWS data
- [x] Root cause clearly marked as likely, not independently confirmed

---

This is an independent learning and research note. It is not official AWS guidance or AWS Support confirmation.
