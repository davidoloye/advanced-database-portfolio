# Portfolio 5 – Distributed Cloud (Google Cloud)

## Objective
Deploy the fintech PostgreSQL database to Google Cloud SQL, provision network and IAM resources with Terraform, and demonstrate a multi‑region read‑replica configuration for high availability.

## Architecture Diagram
> **Note:** Insert a diagram image (e.g., `architecture.png`) in the `screenshots/` folder and view it with the sidecar link `sidecar://...`.

```
[Client] --> Cloud Load Balancer --> Cloud SQL Primary (us-central1)
                                   ↳ Cloud SQL Replica (europe-west1)
```

## Terraform Configuration (excerpt)
```hcl
provider "google" {
  project = var.project_id
  region  = "us-central1"
}

resource "google_sql_database_instance" "primary" {
  name             = "fintech-primary"
  database_version = "POSTGRES_15"
  region           = "us-central1"
  settings {
    tier = "db-f1-micro"
    ip_configuration {
      ipv4_enabled    = true
      require_ssl      = true
    }
  }
}

resource "google_sql_database_instance" "replica" {
  name                = "fintech-replica"
  master_instance_name = google_sql_database_instance.primary.name
  region               = "europe-west1"
  replica_configuration {
    failover_target = false
  }
}

resource "google_sql_user" "app_user" {
  name               = var.db_user
  instance           = google_sql_database_instance.primary.name
  password           = var.db_password
}
```

## Deployment Steps
1. **Install Terraform** – `brew install terraform` (macOS).
2. **Configure GCP credentials** – `gcloud auth application-default login`.
3. **Create `variables.tf`** with `project_id`, `db_user`, `db_password`.
4. Run:
   ```bash
   terraform init
   terraform plan -out=plan.out
   terraform apply "plan.out"
   ```
5. After provisioning, obtain the connection string from the Cloud Console and connect with `psql`.
6. Load the schema from **portfolio‑1** (`psql -f schema.sql`).
7. Verify read‑replica sync by inserting a transaction on the primary and checking the replica.

## Monitoring & Logs
- Enable **Cloud SQL Insights** and **Stackdriver Logging**. Capture logs in `logs/cloudsql.log`.
- Use `EXPLAIN ANALYZE` on replica queries to compare latency.

## Testing
- Run the same queries from **portfolio‑2** against the primary and replica.
- Simulate a primary outage (stop the primary instance) and ensure the replica promotes automatically (record steps in `logs/failover.txt`).

---
### Technical Reflection
See `technical-reflection.md` for discussion on cloud‑native security, Terraform state management, and trade‑offs between managed DB services and self‑hosted solutions.

