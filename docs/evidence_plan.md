# Evidence plan

Describe what you plan to prove, where the evidence will live, and how you will collect it.

### Claim 1
- **Claim:** Storage is reduced by at least 95%.
- **Evidence artifact(s):** S3 Size Report + DynamoDB Item Size.
- **Command or procedure:** Compare the `ContentLength` of the original S3 object with the `ItemSize` of the DynamoDB summary record.
- **Success criteria:** The summary size in DynamoDB is $\le$ 5% of the original S3 file size.

---

### Claim 2
- **Claim:** Key fields (`material_type`, `safety_factor`) are captured accurately.
- **Evidence artifact(s):** Stored JSON Summary + Input Document Excerpt.
- **Command or procedure:** Perform a manual or automated string-match comparison between the "known" values in the test document and the extracted fields in the DynamoDB JSON.
- **Success criteria:** The values in the database exactly match the "known" values from the test document.

---

### Claim 3
- **Claim:** Files are not deleted if the summary is bad/missing.
- **Evidence artifact(s):** CloudWatch Logs + S3 Bucket List (ls).
- **Command or procedure:** Attempt to process a document with missing mandatory fields; check logs for the aborted deletion event and run `aws s3 ls` to verify the object still exists.
- **Success criteria:** Logs show a validation failure and the file remains in S3 (deletion was skipped).
