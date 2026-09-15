# Terraform

Terraform configs for club-managed infrastructure live in
`kuisc/knowledge-base` under `infra/terraform/` (actual `.tf` files, not
docs — this page just explains what's there and how to use it) or in their
own repo if large enough to warrant it.

Document per stack:

- What it provisions and why
- Required provider credentials / how to get access
- `terraform plan`/`apply` instructions
- Teardown instructions (competition infra especially should be easy to tear down)
