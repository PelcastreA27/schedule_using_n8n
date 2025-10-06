# n8n Workflow for Request Approval and Google Calendar Event Creation

##  Description

This **n8n workflow** automates the management of requests (vacations, permissions, etc.) and automatically creates events in **Google Calendar** for approved requests.

The workflow performs the following tasks:

1. Receives requests via a **Webhook**.
2. Sends an email to the approver with **Approve / Reject** buttons.
3. Captures the approver's decision through an **Approval Webhook**.
4. Automatically creates a Google Calendar event if the request is approved.

---

##  Workflow Details

### 1️ Request Webhook
- **URL:** `/webhook/request`
- **Method:** `POST` or `GET`
- **Received variables:**
  - `idForm`: Unique request ID
  - `tipoSol`: Type of request (Vacation, Permission, etc.)
  - `correo_sol`: Requester's email
  - `fechaIni`: Start date
  - `fechaFin`: End date

### 2️ Set Node
- Normalizes and stores necessary variables:
  - `idForm`, `tipoSol`, `correo_sol`, `fechaIni`, `fechaFin`
- Ensures the next nodes have correct data.

### 3️ Send Email Node
- Sends an email to the approver with action buttons:
  - **Approve:**
    ```
    /webhook/approval?id={{$json.idForm}}&status=approved&tipoSol={{$json.tipoSol}}&email={{$json.correo_sol}}&fechaIni={{$json.fechaIni}}&fechaFin={{$json.fechaFin}}
    ```
  - **Reject:**
    ```
    /webhook/approval?id={{$json.idForm}}&status=rejected&tipoSol={{$json.tipoSol}}&email={{$json.correo_sol}}&fechaIni={{$json.fechaIni}}&fechaFin={{$json.fechaFin}}
    ```

### 4️ Approval Webhook
- **URL:** `/webhook/approval`
- **Method:** `GET`
- Receives:
  - `status` → `"approved"` or `"rejected"`
  - `tipoSol`, `correo_sol`, `fechaIni`, `fechaFin`, `idForm`
- **IF Node:**
  - True: `status == "approved"` → continues to Google Calendar
  - False: `status == "rejected"` → optionally send rejection notification

### 5️ Function Node (Optional)
- Converts `fechaIni` and `fechaFin` to **ISO 8601** format:
```javascript
const fechaIniISO = new Date($json.fechaIni + 'T09:00:00-06:00').toISOString();
const fechaFinISO = new Date($json.fechaFin + 'T18:00:00-06:00').toISOString();

return {
  json: {
    ...$json,
    fechaIniISO,
    fechaFinISO
  }
};
