# Claims list

List the claims you intend to prove. Each claim should link to a CCT entry and evidence block.

### Claim 1
- **Claim ID**: 1
- **Clause**: For documents older than 90 days, the system will reduce storage footprint by ≥95% compared to the original file size by replacing full documents with AI summaries.
- **Enforcement point**: S3 Lifecycle policy + DynamoDB summary storage
- **Evidence link**: 
    - S3 object size report before deletion
    - DynamoDB item size for stored summary
    - Lifecycle expiration logs
- **Notes**:

### Claim 2
- **Claim ID**: 2
- **Clause**: AI-generated summaries must contain valid values for the fields:
material_type and safety_factor when those values exist in the source document.
- **Enforcement point**: Bedrock structured JSON prompt + Lambda validation logic
- **Evidence link**:
    - Input document with known values
    - Stored summary JSON in DynamoDB
    - Validation log confirming fields present
- **Notes**:

### Claim 3
- **Claim ID**: 3
- **Clause**: No deletion without valid summary - An original document may be deleted only if its summary passes schema validation and contains all required fields.
- **Enforcement point**: Lambda validation step before delete action
- **Evidence link**:
    - Failed summary test (missing field)
    - Log showing deletion skipped
    - S3 object still present
- **Notes**:

### Claim 4
- **Claim ID**: 4
- **Clause**: Documents tagged with a legal or regulatory hold must be exempt from automated lifecycle deletion, regardless of age. The system must never delete a held document.
- **Enforcement point**: S3 Object Lock or metadata-based hold flag checked by Lambda before any delete action
- **Evidence link**:
    - S3 object with hold tag still present after lifecycle trigger
    - Lambda logs showing deletion skipped due to hold flag
    - Lifecycle policy configuration showing hold-check integration
- **Notes**: Derived from the regulatory retention constraint in the proposal ("some documents may not be legally deletable") and the ethics ledger mitigation for compliance breach risk. This claim ensures the automation respects legal boundaries.

### Claim 5
- **Claim ID**: 5
- **Clause**: AI-generated summaries stored in DynamoDB must be queryable by structured fields (material_type, safety_factor, project_id) so that future engineers can retrieve relevant knowledge without access to the original documents.
- **Enforcement point**: DynamoDB table schema with Global Secondary Indexes (GSIs) on key fields + query validation in Lambda
- **Evidence link**:
    - DynamoDB table description showing GSI definitions
    - Query results returning correct summaries filtered by material_type or safety_factor
    - Comparison of query response time vs. scanning the full table
- **Notes**: Derived from the proposal's stakeholder analysis — future engineers need "structured lessons learned without searching massive files." This claim proves the system does not just delete, it makes the retained knowledge accessible.