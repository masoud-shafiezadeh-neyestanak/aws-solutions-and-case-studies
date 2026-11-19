# AWS re:Post Solutions

This directory contains independently written technical research notes based on selected AWS re:Post discussions.

The purpose is not to reproduce community threads. It is to document what I learned from investigating a technical question, identify the evidence behind a conclusion, and preserve a useful reference for future study.

## Content Principles

- Write an original summary of the technical scenario in your own words.
- Link to the original public re:Post thread instead of copying its question or other users' answers.
- Do not reproduce full posts, comments, screenshots, avatars, or other users' wording.
- Revisit relevant AWS documentation and cite the exact pages that support technical claims.
- Clearly separate documented facts, independent test results, accepted answers, expert-reviewed answers, and unverified hypotheses.
- Do not imply that a community answer or an expert review is an official AWS statement.
- Remove account IDs, personal information, private workload details, credentials, and other sensitive data.
- When discussing an answer you authored, preserve the substance but feel free to improve the explanation and add later findings.

## Participation Snapshot

A dated summary of re:Post participation is available in [`participation-snapshot-2026-11-19.md`](./participation-snapshot-2026-11-19.md). These figures are community-platform outcomes, not official AWS endorsements.

## Selected Research Notes

- [Case 001 — EC2 Reward Activity Not Recognized After Instance Launch](./case-001-ec2-reward-activity-not-recognized.md) — Investigates the difference between an EC2 resource launch, Explore AWS activity completion, and credit posting. The public evidence does not establish the root cause.

## Suggested File Naming

Use one Markdown file per useful research note:

`case-001-short-topic-name.md`

Increase the case number for each new entry. Keep the filename short and descriptive.

## Recommended Workflow

1. Select a re:Post discussion that contains a reusable technical learning point.
2. Summarize the scenario without copying the original wording.
3. Research the relevant AWS documentation and other reliable technical sources.
4. Separate confirmed facts from assumptions and possible explanations.
5. Record the proposed approach, limitations, and validation status.
6. Add the original discussion link and supporting references.
7. Review the note for accuracy, privacy, attribution, and clarity before publishing.

## Validation Labels

Use only labels that accurately describe the evidence:

- **Documented:** Supported by the cited official documentation.
- **Independently tested:** Reproduced or verified in a test environment; describe the test scope.
- **Answer posted:** An answer was published by the repository owner on the linked re:Post thread.
- **Accepted:** The question author marked the answer as accepted.
- **Expert reviewed:** The answer received the relevant expert-review status on re:Post.
- **Not independently tested:** The explanation is based on research and has not been reproduced.

Acceptance or expert review on re:Post should be reported as community-platform status, not as an AWS warranty or official certification of the solution.

## Template

Start from [`_template.md`](./_template.md) when creating a new research note.

---

These notes are independent learning and research materials. They are not official AWS documentation or AWS Support guidance.

## Policy and Licensing References

- [AWS re:Post Terms of Use](https://repost.aws/terms)
- [AWS re:Post Community Guidelines](https://repost.aws/guidelines)
- [AWS Site Terms](https://aws.amazon.com/terms/)

AWS documentation hosted on `docs.aws.amazon.com` is identified in the AWS Site Terms as licensed under **CC-BY-SA-4.0**; code in that documentation is identified as **MIT-0**. When reusing any material rather than writing an independent summary, review the applicable license and provide the required attribution.

For this repository, the preferred approach is to explain technical findings in original wording and link to the relevant source rather than reproduce source material.
