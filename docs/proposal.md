# Capstone proposal

## Problem framing
**The Core Problem**: Engineering organizations generate massive technical documents (CAD exports, compliance reports, design specifications) that are:
- Expensive to store long term in cloud systems
- Rarely accessed in full after a project ends
- Still valuable for knowledge, such as safety limits, materials used, and lessons learned

Companies face a tension between *cost efficiency* (not storing huge files forever) and *knowledge preservation* (not losing critical engineering insights)

Most systems solve this by keeping everything, which is financially inefficient and does not distinguish between raw data and reusable knowledge.

This project proposes a system that preserves technical knowledge while intentionally “forgetting” raw documents after extracting essential information.

## Stakeholders and empty chair
- **Engineering firms**: Reduced long-term cloud storage costs
- **Future engineers/project teams:** Access to structured “lessons learned” without searching massive files
- **Cloud administrators**: Automated lifecycle management instead of manual archiving
- **Compliance & safety teams**: Retention of key safety and material specifications in searchable form
- **Organization leadership**: Clear cost–knowledge trade-off with measurable savings

## Data and privacy
**Types of data the system processes**: Engineering drawings (PDFs), Technical Specifications and safety-related parameters.
However, they may contain design details, trade secrets and compliance sensitive information.

Privacy Considerations: 
- Data is stored in secure cloud storage (S3) with access control.
- AI processing is performed within managed cloud services
- Original files are deleted according to automation retention and minimization policy to reduce exposure risk - the system reduces long-term privacy and security risk by not storing sensitive raw documents indefinitely.

## Constraints and trade-offs
- Limited storage budget: forces deletion of large original files
- AI summarization may be imperfect: Risk of losing technical details
- Automation requirement: less human review of what gets deleted
- Regulatory retention rules: some documents may not be legally deletable
- Cost vs reliability: keeping everything is safe but expensive; summarizing is cheap but lossy

## Intended claims (summary)
- Storage Reduction Guarantee
- Critical Variable Retention
- No Deletion without valid summary