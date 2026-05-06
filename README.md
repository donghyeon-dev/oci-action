# OCI A1 GitHub Actions Bot

This repository runs a scheduled GitHub Actions workflow that tries to launch one Oracle Cloud Infrastructure Always Free A1 instance.

The workflow is based on `oci-a1-github-actions-plan.html` and is designed to:

- run every 15 minutes on a fresh GitHub-hosted runner,
- check whether a running `VM.Standard.A1.Flex` instance already exists,
- launch `dh-server-a1` only when no running A1 instance exists,
- keep retrying quietly on capacity failures,
- send Telegram messages only for important states,
- disable itself after a successful launch or an OCI limit error.

## Files

- `.github/workflows/launch.yml` - scheduled and manually triggerable launch workflow.
- `.github/workflows/upscale.yml` - scheduled and manually triggerable workflow that resizes the existing A1 instance to the Always Free maximum (4 OCPU / 24 GB) or a custom target.
- `docs/secrets.md` - required GitHub repository secrets.
- `oci-a1-github-actions-plan.html` - original implementation plan.

## Setup

1. Create an OCI API key and collect the user OCID, tenancy OCID, fingerprint, private key content, and region.
2. Create or identify the subnet, Ubuntu ARM image, availability domain, and SSH public key to use.
3. Add all required repository secrets from `docs/secrets.md`.
4. Open the GitHub repository Actions tab and run **Launch OCI A1 Instance** manually once.
5. Confirm the workflow reaches either an OCI capacity failure or a successful launch.

The scheduled workflow will continue every 15 minutes until it succeeds or disables itself on a limit error.

## Upscaling to the Always Free maximum

After the launch workflow has created the instance at the safe minimum (1 OCPU / 6 GB), the **Upscale OCI A1 Instance** workflow resizes it to the Always Free maximum.

- Runs every 30 minutes on schedule and is also manually triggerable from the Actions tab.
- Defaults to `ocpus=4` and `memory_in_gbs=24`, which fully consumes the Always Free A1 quota for a single instance. Manual runs accept overrides through workflow inputs.
- Looks up the existing `VM.Standard.A1.Flex` instance in `OCI_COMPARTMENT_ID`, calls `oci compute instance update --shape-config`, then polls until the new shape is reported and the instance is back in `RUNNING`.
- Disables itself after a successful resize, when the instance is already at the target shape, or on a `LimitExceeded` error. Capacity and rate-limit failures are retried on the next schedule.
- Sends Telegram notifications mirroring the launch workflow: success with before/after shape, retryable failures with a reason hint, and permanent disable on `LimitExceeded`.

OCI applies Flex shape changes online when possible and otherwise reboots the instance, so expect a brief reboot and short SSH downtime.

GitHub Actions only runs scheduled workflows from the repository's default branch, so the cron schedule activates once `upscale.yml` is merged into the default branch.

## Security

Never commit OCI private keys, Telegram tokens, `.env` files, or local OCI CLI config files. They belong only in GitHub repository secrets.

