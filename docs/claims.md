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
- **Claim ID**:
- **Clause**:
- **Enforcement point**:
- **Evidence link**:
- **Notes**:

### Claim 5
- **Claim ID**:
- **Clause**:
- **Enforcement point**:
- **Evidence link**:
- **Notes**: