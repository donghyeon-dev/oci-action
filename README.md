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
- `.github/workflows/upscale.yml` - manually triggered workflow that resizes the existing A1 instance to the Always Free maximum (4 OCPU / 24 GB) or a custom target.
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

After the launch workflow has created the instance at the safe minimum (1 OCPU / 6 GB), trigger **Upscale OCI A1 Instance** manually from the Actions tab to resize it to the Always Free maximum.

- Defaults to `ocpus=4` and `memory_in_gbs=24`, which fully consumes the Always Free A1 quota for a single instance.
- Looks up the existing `VM.Standard.A1.Flex` instance in `OCI_COMPARTMENT_ID`, calls `oci compute instance update --shape-config`, then polls until the new shape is reported and the instance is back in `RUNNING`.
- Sends a Telegram message with the before/after shape on success, and a failure notice (with reason hint for `LimitExceeded`, capacity, or rate-limit errors) on failure.

OCI applies Flex shape changes online when possible and otherwise reboots the instance, so expect a brief reboot and short SSH downtime.

## Security

Never commit OCI private keys, Telegram tokens, `.env` files, or local OCI CLI config files. They belong only in GitHub repository secrets.

