Security Research: Vodafone Oman Pre-Order API Pricing Logic Flaw

Overview & Metadata
- Target Organization: Vodafone Oman (www.vodafone.om)[span_0](start_span)[span_0](end_span)
- Vulnerable Endpoint: POST /api/backend/ajax/v1/preorders[span_1](start_span)[span_1](end_span)
- Vulnerability Type: Business Logic Flaw / Missing Server-Side Validation[span_2](start_span)[span_2](end_span)
- CWE Category: CWE-840 (Business Logic Errors)
- HackerOne Report ID: #3880372[span_3](start_span)[span_3](end_span)
- Status: Resolved (Deprecated Endpoint)[span_4](start_span)[span_4](end_span)

Executive Summary
A critical business logic vulnerability was identified in the pre-order processing API. The application relied on client-supplied pricing parameters (deviceCost, VAT, totalCost) extracted directly from the DOM without server-side recalculation[span_5](start_span)[span_5](end_span). This allowed the submission of arbitrary, zero, or negative pricing values[span_6](start_span)[span_6](end_span).

Proof of Concept (PoC)

Frontend Parameter Exposure
The frontend script (/wp-content/themes/vodafone/js/custom_preorder.min.js) sends DOM pricing data directly without integrity protection[span_7](start_span)[span_7](end_span):
var e = {
    orderId: uuidv4(),
    device: h.value,
    color: B.value,
    capacity: F.value,
    deviceCost: $("#deviceCost").text(),
    VAT: $("#VAT").text(),
    totalCost: $("#totalCost").text(),
    fullName: l.val().trim(),
    contactNumber: o.val().trim(),
    type: "pre-order"
};

Server Response Validation
- Zero Price (deviceCost=0): Accepted with 200 OK (true)[span_8](start_span)[span_8](end_span).
- Negative Price (deviceCost=-500): Accepted with 200 OK (true)[span_9](start_span)[span_9](end_span).
- Non-numeric (deviceCost=abc): Accepted with 200 OK (true)[span_10](start_span)[span_10](end_span).

Summary Table

| Parameter | Test Value | Server Response |
| :--- | :--- | :--- |
| deviceCost | 0 | true (Accepted)[span_11](start_span)[span_11](end_span) |
| deviceCost | -500 | true (Accepted)[span_12](start_span)[span_12](end_span) |
| deviceCost | 999999 | true (Accepted)[span_13](start_span)[span_13](end_span) |
| deviceCost | abc | true (Accepted)[span_14](start_span)[span_14](end_span) |
| contactNumber | (Empty) | false (Rejected)[span_15](start_span)[span_15](end_span) |

Remediation
1. Server-Side Re-calculation: Never trust client-side pricing data; always fetch authoritative prices from the backend database.
2. Strict Input Validation: Enforce proper data types and ranges for all incoming parameters.
