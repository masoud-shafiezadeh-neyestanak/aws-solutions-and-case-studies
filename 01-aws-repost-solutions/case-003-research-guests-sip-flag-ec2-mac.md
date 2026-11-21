# Case 003 — Research Guest SIP Flag on Apple Silicon EC2 Mac

> An independent technical research note based on an AWS re:Post discussion. The scenario and analysis are summarized in original wording; the original question and community answer are not reproduced.

## 1. Source Discussion

- **Platform:** AWS re:Post
- **Discussion:** [Enabling Research Guests on an Apple Silicon EC2 Mac](https://repost.aws/questions/QUq3txFCKOQ0ad5M4jZQh6uA/enabling-research-guests-on-an-apple-silicon-ec2-mac#ANm1F-BtTrTB20eQmOoaifpg)
- **Date reviewed:** 2026-11-21
- **My participation:** Answer published in the discussion
- **Thread outcome:** Answer accepted by the question author and marked Expert Approved on re:Post

These labels describe the status of the answer on the re:Post platform. They do not constitute an official AWS guarantee or confirmation that the requested configuration is supported.

## 2. Technical Scenario

The question concerned an Apple silicon Amazon EC2 Mac instance and whether the SIP-related flag `CSR_ALLOW_RESEARCH_GUESTS` could be enabled on its boot volume.

The author had identified two possible routes:

- Configure the flag through the EC2 Mac SIP modification API.
- Start macOS Recovery and attempt to run `csrutil allow-research-guests`.

The specific flag was not present among the SIP settings exposed by the EC2 API. The central issue was therefore not simply whether macOS has a command for the setting, but whether AWS documents and supports changing it on an EC2 Mac instance.

## 3. AWS Services and Technical Context

- **Compute service:** Amazon EC2
- **Instance platform:** Apple silicon Mac
- **Security feature:** macOS System Integrity Protection (SIP)
- **EC2 API:** `CreateMacSystemIntegrityProtectionModificationTask`
- **Related API data type:** `MacSystemIntegrityProtectionConfigurationRequest`
- **Requested SIP flag:** `CSR_ALLOW_RESEARCH_GUESTS`
- **Proposed macOS command:** `csrutil allow-research-guests`

## 4. Key Technical Questions

1. Which SIP settings can AWS configure through the EC2 Mac SIP modification API?
2. Is `CSR_ALLOW_RESEARCH_GUESTS` one of the supported API configuration options?
3. Does AWS document a supported procedure for changing this flag through macOS Recovery on an EC2 Mac instance?
4. Can behaviour documented for a physical Apple silicon Mac be assumed to be supported on an EC2 Mac instance?
5. What is the safest next step when a workload requires a SIP setting that is not exposed by the EC2 API?

## 5. Investigation and Evidence

The official EC2 API reference and user guide establish the following:

| Finding | Evidence | Confidence |
|---|---|---|
| AWS provides an API to create SIP modification tasks for x86 and Apple silicon EC2 Mac instances. | EC2 API — CreateMacSystemIntegrityProtectionModificationTask | Documented |
| The API supports an overall SIP status and selective overrides for individual settings. | EC2 API — CreateMacSystemIntegrityProtectionModificationTask | Documented |
| The selective configuration request exposes seven settings: Apple Internal, Base System, Debugging Restrictions, DTrace Restrictions, Filesystem Protections, Kext Signing, and NVRAM Protections. | EC2 API — MacSystemIntegrityProtectionConfigurationRequest | Documented |
| `CSR_ALLOW_RESEARCH_GUESTS` is not listed as a field in the documented selective configuration request. | EC2 API — MacSystemIntegrityProtectionConfigurationRequest | Documented |
| On Apple silicon EC2 Mac instances, SIP configuration is applied at the volume level. A newly attached root volume does not inherit the previous volume's SIP configuration. | EC2 User Guide — Configure System Integrity Protection for Amazon EC2 Mac instances | Documented |
| AWS recommends verifying custom SIP settings after applying a configuration and reviewing them after macOS upgrades. | EC2 User Guide — Configure System Integrity Protection for Amazon EC2 Mac instances | Documented |

### Evidence Boundary

The absence of `CSR_ALLOW_RESEARCH_GUESTS` from the documented EC2 API configuration is clear. The public AWS documentation reviewed for this note does not describe a supported EC2 Mac procedure for enabling this specific flag through macOS Recovery.

That does **not** prove that no low-level or technically possible method exists in macOS. It means that this research found no AWS-documented, supported procedure for this flag on EC2 Mac.

## 6. Analysis

The EC2 SIP modification API is not a general-purpose interface for setting arbitrary macOS boot-policy or SIP flags. It accepts an overall SIP status and a defined set of selective configuration fields.

Because `CSR_ALLOW_RESEARCH_GUESTS` is not among those documented fields, it cannot be requested as one of the selective settings through the documented `CreateMacSystemIntegrityProtectionModificationTask` API.

The Recovery-mode question needs a separate qualification. A command or workflow that may be available on a physical Apple silicon Mac should not automatically be treated as an AWS-supported procedure on an EC2 Mac instance. The AWS documentation reviewed here describes the EC2 SIP modification workflow, but does not establish that manually changing this flag in macOS Recovery is supported for EC2 Mac.

For Apple silicon, the volume-level behaviour is particularly important: AWS applies SIP configuration to a volume, and a replacement or newly attached root volume does not simply inherit another volume's SIP configuration.

## 7. Recommended Approach

1. **Check the documented API surface.** Review the current `MacSystemIntegrityProtectionConfigurationRequest` fields before designing an automation or operational procedure.
2. **Do not substitute a different SIP option.** Disabling overall SIP or changing one of the seven available settings is not evidence that the requested research-guest flag has been enabled.
3. **Avoid assuming Recovery-mode equivalence.** Do not rely on instructions for a physical Mac as proof that the same procedure is supported on EC2 Mac.
4. **Ask AWS Support for a supportability decision.** Explain the workload requirement, the exact flag, the Apple silicon EC2 Mac instance family, macOS version, and the intended boot-volume workflow. Ask specifically whether AWS supports enabling `CSR_ALLOW_RESEARCH_GUESTS` on that EC2 Mac configuration.
5. **Use a supported alternative if the requirement is not supported.** If AWS confirms that this flag cannot be configured on EC2 Mac, evaluate a different development or testing environment whose platform explicitly supports the required capability.

No undocumented boot-policy changes should be attempted on a production or important environment solely on the assumption that a physical Mac procedure will also work on EC2.

## 8. Validation Status

- [x] EC2 SIP configuration user guide reviewed
- [x] EC2 SIP modification API reference reviewed
- [x] Supported selective SIP configuration fields checked
- [ ] Recovery-mode procedure independently tested on EC2 Mac
- [ ] AWS Support confirmation obtained for this specific flag
- [x] Answer posted in the linked re:Post discussion
- [x] Answer accepted by the question author
- [x] Answer marked Expert Approved on re:Post

**Conclusion:** The documented EC2 SIP modification API does not expose `CSR_ALLOW_RESEARCH_GUESTS`. No AWS-documented Recovery-mode procedure for enabling this flag on EC2 Mac was identified in the sources reviewed. Whether AWS supports any other method should be confirmed directly with AWS Support.

## 9. What I Learned

- A cloud provider's supported control-plane API may expose only a subset of the configuration options available in the underlying operating system.
- An API's documented parameter schema is an important boundary when assessing whether a configuration is supported.
- Apple silicon EC2 Mac applies SIP configuration at the volume level, which matters when replacing or attaching root volumes.
- A technically plausible command is not the same as a provider-supported operational procedure.
- Good technical answers distinguish documented support, possible platform behaviour, and unknowns that require vendor confirmation.

## 10. References

1. [Configure System Integrity Protection for Amazon EC2 Mac instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/mac-sip-settings.html) — AWS workflow, Apple silicon volume-level behaviour, prerequisites, and supported configuration process.
2. [CreateMacSystemIntegrityProtectionModificationTask](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_CreateMacSystemIntegrityProtectionModificationTask.html) — EC2 API operation for creating SIP modification tasks.
3. [MacSystemIntegrityProtectionConfigurationRequest](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_MacSystemIntegrityProtectionConfigurationRequest.html) — Exact list of selective SIP configuration fields exposed by the API.
4. [About System Integrity Protection](https://support.apple.com/en-us/102149) — Apple's general explanation of macOS System Integrity Protection.
5. [Original AWS re:Post discussion](https://www.repost.aws/questions/QUq3txFCKOQ0ad5M4jZQh6uA/enabling-research-guests-on-an-apple-silicon-ec2-mac) — Community discussion and its platform status.

## 11. Privacy and Attribution Review

- [x] Scenario summarized in original wording
- [x] Other users' full question and answer text not reproduced
- [x] No account-specific or personal information included
- [x] AWS and Apple references linked
- [x] Accepted and Expert Approved statuses attributed to re:Post
- [x] No unsupported claim that AWS supports an undocumented Recovery-mode procedure

---

This is an independent learning and research note. It is not official AWS guidance or AWS Support confirmation.
