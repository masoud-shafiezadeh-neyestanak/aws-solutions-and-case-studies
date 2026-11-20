# Case 001 — EC2 Reward Activity Not Recognized After Instance Launch

> An independent research note based on an AWS re:Post discussion. This document summarizes the technical scenario and the investigation in original wording; it does not reproduce the original question or community answer.

## 1. Source Discussion

- **Platform:** AWS re:Post
- **Discussion URL:** Not yet provided; add the exact public re:Post link when available.
- **Date reviewed:** 2026-11-19
- **My participation:** Answer published in the discussion
- **Thread outcome:** Answer accepted by the question author and marked Expert Approved on re:Post

The acceptance and review labels describe the status shown by the re:Post community platform. They are not an official AWS guarantee or confirmation that the underlying account issue was resolved.

## 2. Technical Scenario

A new AWS customer reported that the **Launch an instance using Amazon EC2** activity in the **Explore AWS** area remained marked as **In Progress**, even after successfully launching EC2 instances.

The reported activity was worth USD 20. The account had already received USD 80 out of the possible USD 100 in additional activity credits. The user reported two `t3.micro` launches in the Asia Pacific (Sydney) Region (`ap-southeast-2`): one instance remained running, while the other was later terminated. The activity had not changed status after more than 24 hours.

The central question was whether creating an EC2 instance outside the activity's guided flow is sufficient for the Explore AWS activity to be recognized as complete.

## 3. AWS Services and Context

- **AWS account experience:** New AWS sign-up / Free Tier activity experience
- **Console area:** Explore AWS widget
- **Activity:** Launch an instance using Amazon EC2
- **Service:** Amazon Elastic Compute Cloud (Amazon EC2)
- **Reported instance family and size:** `t3.micro`
- **Reported Region:** `ap-southeast-2`
- **Reported reward:** USD 20
- **Reported additional activity-credit balance:** USD 80 of USD 100

No account ID, instance ID, personal information, or other account-specific identifier is reproduced in this note.

## 4. Key Technical Questions

1. Does launching an EC2 instance directly from the EC2 console satisfy the corresponding Explore AWS activity, or must the learner complete a guided activity flow?
2. Is the activity still within its eligibility period and displayed completion deadline?
3. Is the AWS account still eligible to earn additional activity credits?
4. Has the credit appeared on the Billing and Cost Management Credits page, even if the Explore AWS activity status has not refreshed?
5. What evidence should be provided to AWS Support if the activity remains incomplete?

## 5. Investigation and Evidence

The AWS documentation establishes the following:

| Finding | Evidence | Confidence |
|---|---|---|
| Eligible new customers can earn USD 20 for each completed activity, up to USD 100 in additional activity credits. | AWS Billing documentation: Earning additional credits | Documented |
| The activities are presented in the Explore AWS widget on the AWS Console Home dashboard. | AWS Billing documentation: Earning additional credits | Documented |
| The EC2 activity is intended to guide a customer through launching an instance and learning how to clean it up. | AWS Billing documentation: Launch an instance using Amazon EC2 | Documented |
| The EC2 activity can incur AWS service charges, which are deducted from applicable Free Tier credits. | AWS Billing documentation: Earning additional credits | Documented |
| Activities must be completed by their specified deadline and within the applicable eligibility period. Joining an AWS Organization or setting up an AWS Control Tower landing zone can make activities expire immediately. | AWS Free Tier FAQs | Documented |
| Additional activity credits are not available after advanced features are activated. | AWS Free Tier FAQs | Documented |
| AWS states that credits may take up to 30 minutes to appear on the Credits page after earning them. | AWS Free Tier FAQs | Documented |

### Important Evidence Boundary

The public AWS documentation describes the EC2 activity and directs customers to the Explore AWS widget. However, the documentation reviewed for this note does **not** explicitly specify whether an EC2 instance launched independently through the standard EC2 console is always sufficient to complete the tracked activity.

Therefore, the guided-workflow possibility is a troubleshooting check, not a confirmed explanation of the reported behavior.

The documented delay of up to 30 minutes concerns credits appearing on the Credits page. It should not be treated as a documented refresh-time guarantee for the Explore AWS activity's **In Progress** status.

## 6. Analysis

The existence of a successfully launched EC2 instance does not, by itself, prove that the separate Explore AWS activity-tracking workflow has registered completion.

The distinction to investigate is between:

- **Resource operation:** An EC2 instance was successfully created in the account.
- **Learning activity completion:** The Explore AWS activity registered the required steps as completed.
- **Credit posting:** The earned credit appeared in the account's Credits page.

These are related, but the public documentation does not establish that they update at exactly the same time or through the same mechanism.

Because the status reportedly remained unchanged for more than 24 hours, the documented credit-display delay of up to 30 minutes would not, on its own, explain the entire observation. However, the public evidence is insufficient to identify the root cause.

## 7. Proposed Troubleshooting Approach

1. **Reopen the activity in Explore AWS.** From the AWS Console Home dashboard, locate the Explore AWS widget and filter for activities that earn AWS credits. Open the EC2 activity and review its own instructions and completion deadline.
2. **Check the guided steps.** Compare the actions already completed with the steps presented by the activity. Do not assume that a separately launched instance automatically satisfies every tracked step; AWS's public documentation does not explicitly confirm this.
3. **Confirm account eligibility.** Verify that the activity has not passed its specified completion date and that the account has not entered a state that makes further activity credits unavailable, such as joining an AWS Organization, setting up an AWS Control Tower landing zone, or activating advanced features, where applicable.
4. **Check the Credits page separately.** Confirm whether the USD 20 credit appears in Billing and Cost Management, rather than relying only on the activity's progress label.
5. **Escalate with evidence if unresolved.** Contact AWS Support through the available account or billing support path. Provide the relevant account and resource identifiers privately, together with launch timestamps, Region, the activity name, and screenshots of its status. Do not publish account-specific identifiers in a public discussion or this repository.

Avoid launching additional instances solely to try to trigger the reward. The activity can incur service charges, and the available documentation does not promise that repeated launches will resolve an activity-tracking problem.

## 8. Validation Status

- [x] Relevant AWS documentation reviewed
- [ ] Root cause independently reproduced
- [ ] Credit award independently verified
- [x] Answer posted in the linked re:Post discussion
- [x] Answer accepted by the question author
- [x] Answer marked Expert Approved on re:Post
- [ ] Final resolution confirmed in the supplied discussion details

**Resolution status:** The answer was accepted and marked Expert Approved, but the information available for this research note does not confirm whether the USD 20 credit was ultimately awarded or what caused the activity to remain in progress.

## 9. What I Learned

- A successful service operation and completion of a console-based learning activity should be treated as separate states until the activity itself confirms completion.
- Credit posting and activity-progress tracking are not necessarily the same status indicator.
- AWS Free Tier activity credits have eligibility conditions and deadlines that should be checked before repeating a potentially billable action.
- A useful troubleshooting response can provide a safe diagnostic path and an evidence checklist without claiming to know an undocumented root cause.
- Accepted and Expert Approved community answers are useful quality signals, but the technical conclusion must still be bounded by official documentation and available evidence.

## 10. References

1. [AWS Billing — Earning additional credits](https://docs.aws.amazon.com/us_en/awsaccountbilling/latest/aboutv2/free-tier-plans-activities.html) — Activity value, Explore AWS location, EC2 activity purpose, and service-charge warning.
2. [AWS Free Tier FAQs](https://aws.amazon.com/free/free-tier-faqs/) — Activity eligibility, completion deadlines, ineligibility conditions, and the stated credit-display delay.
3. [AWS Billing — Explore AWS services with AWS Free Tier](https://docs.aws.amazon.com/us_en/awsaccountbilling/latest/aboutv2/free-tier.html) — Free and Paid account plan context.
4. [Original AWS re:Post discussion](https://www.repost.aws/questions/QUYAdrSqeRSA-rqs4GeY4SXA/aws-new-reward-credit-not-applied-launch-an-instance-using-ec-2-still-showing-in-progress) — Community discussion and its platform status.

## 11. Privacy and Attribution Review

- [x] Scenario summarized in original wording
- [x] Other users' full question and answer text not reproduced
- [x] Personal and account-specific identifiers omitted
- [x] Official documentation linked
- [x] Accepted and Expert Approved statuses attributed to re:Post
- [x] No unsupported root cause or AWS endorsement claimed

---

This is an independent learning and research note. It is not official AWS guidance or AWS Support confirmation.
