# Week 9 Security Audit — cis410-deploy-sa

**Project:** cis410-yalew

**Date:** June 1, 2026

**Auditor:** Welelategegn

---

## 1. IAM Audit Results

### Before — Week 8 Configuration (over-permissioned)

| Role | Scope | Problem |
|---|---|---|
| roles/run.admin | Project | Overly broad — grants ability to delete services and modify IAM, not just deploy |
| roles/storage.admin | Project | Overly broad — grants access to ALL GCS buckets in the project |
| roles/artifactregistry.writer | Project | Acceptable — scoped to push images only |
| roles/viewer | Project | Acceptable — read-only project metadata |
| roles/iam.serviceAccountUser | Compute SA | Required — needed to act as Compute Engine default SA |

### After — Week 9 Least-Privilege Fix

| Role | Scope | Why Sufficient |
|---|---|---|
| roles/run.developer | Project | Deploy only — cannot delete services or modify IAM |
| roles/storage.admin | tfstate bucket only | Scoped to one bucket — not all storage |
| roles/artifactregistry.writer | Project | Unchanged — push images only |
| roles/viewer | Project | Unchanged — read project metadata |
| roles/iam.serviceAccountUser | Compute SA | Unchanged — required for Cloud Run deployment |

---

## 2. Secret Manager Migration

- Secret created: flask-app-secret
- Replication: automatic
- Access granted to: cis410-deploy-sa
- Access granted to: Compute Engine default service account
- Cloud Run update: APP_SECRET environment variable mounted from Secret Manager

---

## 3. Monitoring Configuration

- Log-based alert: cis410-flask-app-errors
- Billing budget: cis410-monthly-budget — $20 limit, alerts at 50%, 90%, and 100%

---

## 4. Reflection

### Q1: Why is roles/run.admin inappropriate for a CI/CD pipeline service account?

The roles/run.admin permission grants more access than a deployment pipeline needs. It can modify or delete Cloud Run services and perform administrative actions. Using roles/run.developer follows the principle of least privilege and reduces security risks.

### Q2: What is the security difference between storing a secret in GitHub Secrets vs. Google Secret Manager?

GitHub Secrets are used during workflow execution and are stored within GitHub. Google Secret Manager provides centralized storage, access control, and auditing within Google Cloud. This makes it easier to manage and protect sensitive information.

### Q3: What is the risk of delaying IAM cleanup until after launch?

Leaving excessive permissions in place increases the chance of accidental changes or unauthorized access. If an account is compromised, an attacker could gain broader access to resources. Applying least privilege early helps reduce these risks.
