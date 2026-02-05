# Clause -> Control -> Test (CCT) v1

| Clause | Control | Test | Enforcement point | Evidence |
| --- | --- | --- | --- | --- |
| 1 | S3 Lifecycle rule deletes original object after 90 days; Lambda stores structured summary in DynamoDB. | Upload a known-size file. After lifecycle execution (or simulated trigger), measure: (1) original S3 object no longer exists, (2) summary record size in DynamoDB ≤5% of original. | S3 Lifecycle configuration + Lambda storage logic. | S3 object metadata (size + deletion log), DynamoDB item size, lifecycle policy JSON. |
| 2 | Bedrock prompt enforces structured JSON schema; Lambda validates presence and format of required fields before saving. | Upload a document containing known values. Query stored summary and verify exact values appear in the structured output. | Lambda validation logic before DynamoDB write. | Source document excerpt, stored JSON summary, validation log output. |
| 3 | Lambda conditional logic: delete action executes only after validation flag = success. Failed summaries halt deletion. | Force a failure (e.g., remove safety_factor from AI output or simulate API failure). Confirm original S3 object remains and deletion does not occur. | Lambda decision branch controlling S3 delete call. | CloudWatch logs showing validation failure, absence of S3 delete event, object still present in bucket. |
| 4 | ... | ... | ... | ... |
| 5 | ... | ... | ... | ... |
