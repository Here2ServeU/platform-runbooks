# Red Hat Training Access Runbook

## Purpose

Use this runbook to request access to the Red Hat Partner Training Portal and associated OpenShift courses through the approved partner support process.

## Scope

This runbook is intended for engineers who:

- are enrolled in Red Hat or OpenShift training through an internal learning portal
- cannot access course content directly
- need partner-linked training access rather than a standalone personal account

## Prerequisites

- active company email address
- internal enrollment or manager approval for the course
- course ID or course title, for example `DO180` or `DO280`
- access to the Partner Acceleration Desk form

## Standard Request Path

Open:

```text
https://connect.redhat.com/support/partner-acceleration-desk
```

Recommended form values:

### Sub Category

```text
Skills training
```

### Request Summary

```text
Access to Red Hat Partner Training Portal for OpenShift Training
```

## Sanitized Request Templates

### Standard Template

```text
Hello Team,

I am an employee of <company-name> requesting access to the Red Hat Partner Training Portal in order to complete OpenShift training courses.

Specifically, I am working on:
- Red Hat OpenShift Administration II: Configuring a Production Cluster (DO280)

I attempted to access the training through my company learning platform, but I understand that access must be granted through Red Hat Partner Connect.

I have reviewed the Partner Training Portal Quick Start Guide and followed the steps to request access to an existing partner account.

Could you please grant me access so I can proceed with my training?

Please let me know if any additional information is required.

Thank you,
<your-name>
```

### Engineer-Focused Template

```text
Hello Team,

I am currently working in an OpenShift platform engineering role and require access to the Red Hat Partner Training Portal to continue hands-on training.

I am enrolled in:
- Red Hat OpenShift Administration II (DO280)

This training supports ongoing work related to cluster deployment, GitOps operations, and production platform configuration.

I attempted access through my company learning platform but understand that partner-linked access is required.

Please grant access to the appropriate partner account so I can proceed.

Thank you,
<your-name>
```

## Recommended Execution Steps

1. Confirm the exact course code and title.
2. Confirm your company email address is the one tied to partner access.
3. Submit the Partner Acceleration Desk request.
4. Save the ticket number in your internal tracking system.
5. Wait for approval or follow up if the SLA is exceeded.

## Expected Timeline

| Step | Typical duration |
|---|---|
| Submit request | same day |
| Initial review | same day to 2 business days |
| Access confirmation | email notification |

## Validation Checklist

After approval, validate the following:

- your partner-linked account can sign in
- the assigned courses appear in the training portal
- labs and learning paths open without access errors
- your account is associated with the correct organization

## Best Practices

- use your company email only
- do not create duplicate Red Hat accounts for the same corporate identity
- include the exact course code in the request
- keep the request short, factual, and role-specific
- capture the support case number for follow-up

## Troubleshooting

### Cannot Access Training Content

- confirm the request was submitted through the Partner Acceleration Desk, not a public support path
- verify the course is tied to your company entitlement or training plan
- confirm you are signing in with the same email used in the request

### Already Created A Personal Red Hat Account

- do not create additional accounts
- tell the support team which email address should be linked to the partner organization
- wait for support guidance before retrying registration

### No Response After Two Business Days

Use this follow-up note:

```text
Following up on my request for Red Hat Partner Training access. This access is required for ongoing OpenShift training and platform engineering work. Please let me know if additional details are needed.
```

### Access Granted But Courses Are Missing

- verify the entitlement or training assignment in the internal learning platform
- confirm the correct partner organization was linked
- ask support to verify course visibility for your user profile

### Escalation Data To Capture

Include these details when escalating internally or to support:

- support ticket number
- company email address used
- exact course code
- date of submission
- screenshot of the access error with sensitive data removed
