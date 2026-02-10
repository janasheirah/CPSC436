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

---

### Claim 4
- **Claim:** Documents under legal or regulatory hold are never deleted by the automated lifecycle.
- **Evidence artifact(s):** S3 object metadata + CloudWatch Lambda logs + CloudTrail DeleteObject events.
- **Command or procedure:**
  1. Upload a test document and tag it with `x-amz-meta-legal-hold: true`.
  2. Trigger the deletion Lambda (manually or via lifecycle simulation).
  3. Run `aws s3api head-object --bucket <bucket> --key <key>` to confirm the object still exists.
  4. Check CloudWatch logs for the "hold detected — deletion skipped" entry.
  5. Check CloudTrail for absence of a DeleteObject event for that key.
  6. Remove the hold tag and re-trigger; confirm the object is now deleted.
- **Success criteria:** The held object survives the deletion trigger. Logs explicitly show the hold was detected. After the hold is removed, normal deletion proceeds — proving the hold is the only thing that prevented deletion.

---

### Claim 5
- **Claim:** AI summaries are queryable by structured fields so future engineers can find relevant knowledge.
- **Evidence artifact(s):** DynamoDB table description (GSIs) + Query response JSON + Lambda validation error.
- **Command or procedure:**
  1. Run `aws dynamodb describe-table --table-name <table>` to confirm GSIs exist on `material_type`, `safety_factor`, and `project_id`.
  2. Query the table: `aws dynamodb query --table-name <table> --index-name material_type-index --key-condition-expression "material_type = :v" --expression-attribute-values '{":v":{"S":"steel"}}'`.
  3. Verify results contain only matching summaries.
  4. Call the Lambda query endpoint with no filter key and confirm it returns a 400 validation error.
  5. Measure query latency from CloudWatch metrics for a table with ≥50 items.
- **Success criteria:** GSIs are defined. Filtered queries return correct results. Queries without a filter are rejected. Response time is under 500ms.
