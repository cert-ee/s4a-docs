API Mechanics
===================

JSON format is used for all communication.
Detectors are authenticated to Central API via x-access-token header.
Node npm package: [https://github.com/strongloop/loopback](https://github.com/strongloop/loopback)

**Accepted / required fields are defined in standard loopback configuration files as remote methods 
( /server/common/models )**

## Detector Registration

Detector sends request to register :
No **x-access-token** required as this is the starting point.
```
POST [CENTRAL_API_URL]/api/registration/request:
{
 name: machine_id,
 first_name: first_name,
 last_name: last_name,
 organization_name: organization_name,
 contact_phone: contact_phone,
 contact_email: contact_email,
 machine_id: machine id
}
```
Response:
```
{
 token: token,
 temporary: true
}
```

After CENTRAL has manually approved the registration, a task is placed in the api/report/status response which directs the detector to either Ok or Rejected path:
```
GET [CENTRAL_API_URL]/api/registration/completeRegistration
```
Response:
```
{
 unique_name: unique_name,
 first_name: first_name,
 last_name: last_name,
 organization_name: organization_name,
 contact_phone: contact_phone,
 contact_email: contact_email,
 csr_signed: csr_signed,
 token:
  {
   token: apikey
   temporary: false
    },
 message: "Registration approved”
}
```
Request if registration was not successful
```
GET [CENTRAL_API_URL]/api/registration/completeRegistrationRejected
```
Response:
```
{
 registration_status: "Rejected",
 reject_reason: reject_reason
}
```


## Detector reporting

#### Status
```
POST [CENTRAL_API_URL]/api/report/status
{
 components: [],
 rules: []
}
```
Response ( if there is any tasks to be completed )
```
{
 job_queue: []
}
```
#### Alerts
```
POST [CENTRAL_API_URL]/api/report/alerts
{
 alerts: []
}
```
Response:
```
{
 message: “ok”
}
```

#### Rules
```
POST [CENTRAL_API_URL]/api/report/rules
{
 last_update: DATETIME
}
```
Response:
```
{
 rules: []
}
```

#### Feedback
```
POST [CENTRAL_API_URL]/api/report/feedback
{
 message: message,
 machine_id: machine_id
 logs: logs
}
```
Response
```
{
 case_number: case_number,
 faq_url: FAQ_URL
}
```