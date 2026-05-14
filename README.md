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
- `.github/workflows/monitor-usage.yml` - scheduled and manually triggerable workflow that monitors A1 shape allocation, boot/block volume capacity, monthly VNIC egress, and Usage API cost rows against Always Free thresholds.
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

## Monitoring Always Free usage

The **Monitor OCI Free Tier Usage** workflow checks the active Always Free risk areas on a schedule and on manual dispatch.

- Runs every 6 hours and can be triggered from the Actions tab.
- Reads raw OCI data from Compute, Block Volume, VNIC Monitoring metrics, and Usage API.
- Validates the computed monitor values by printing the raw instance shape rows, raw volume rows, VNIC metric subtotals, and Usage API billing-side rows in the workflow log.
- Computes percentage-based risk for A1 OCPU, A1 memory, boot/block volume capacity, and monthly VNIC egress against Always Free limits.
- Posts a Discord summary on every run when `DISCORD_WEBHOOK_URL` is configured, intended for a webhook created in the `#instance-monitoring` channel.
- Fails the run on critical threshold breaches and sends a Telegram notification for warning or critical states.
- Defaults:
  - A1 allocation critical threshold: more than 4 OCPU or 24 GB memory.
  - Boot + block volume critical threshold: more than 200 GB; warning from 180 GB.
  - Monthly VNIC egress warning: 8 TiB; critical: 10 TiB.
  - Usage API computed amount greater than zero is critical.

The VNIC metric uses `VnicToNetworkBytes` from the `oci_vcn` namespace as a near-real-time traffic signal. The Usage API rows are included as the billing-side cross-check, but they can lag behind live metrics.

To send every monitor result to Discord, create a webhook in the Discord `#instance-monitoring` channel and add its URL as a GitHub Actions secret named `DISCORD_WEBHOOK_URL`. The workflow cannot create the Discord channel or webhook itself; those must be configured in Discord first.

## Security

Never commit OCI private keys, Telegram tokens, `.env` files, or local OCI CLI config files. They belong only in GitHub repository secrets.

