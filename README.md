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
- `docs/secrets.md` - required GitHub repository secrets.
- `oci-a1-github-actions-plan.html` - original implementation plan.

## Setup

1. Create an OCI API key and collect the user OCID, tenancy OCID, fingerprint, private key content, and region.
2. Create or identify the subnet, Ubuntu ARM image, availability domain, and SSH public key to use.
3. Add all required repository secrets from `docs/secrets.md`.
4. Open the GitHub repository Actions tab and run **Launch OCI A1 Instance** manually once.
5. Confirm the workflow reaches either an OCI capacity failure or a successful launch.

The scheduled workflow will continue every 15 minutes until it succeeds or disables itself on a limit error.

## Security

Never commit OCI private keys, Telegram tokens, `.env` files, or local OCI CLI config files. They belong only in GitHub repository secrets.

