# Required GitHub Secrets

Add these under repository **Settings -> Secrets and variables -> Actions -> New repository secret**.

| Secret | Value |
| --- | --- |
| `OCI_CLI_USER` | OCI user OCID from the API key configuration preview. |
| `OCI_CLI_TENANCY` | OCI tenancy OCID. |
| `OCI_CLI_FINGERPRINT` | API key fingerprint, including colons. |
| `OCI_CLI_KEY_CONTENT` | Full private key content, including `BEGIN PRIVATE KEY` and `END PRIVATE KEY` lines. |
| `OCI_CLI_REGION` | OCI region, for example `ap-chuncheon-1`. |
| `OCI_COMPARTMENT_ID` | Target compartment OCID. For root compartment, use the tenancy OCID. |
| `OCI_SUBNET_ID` | Target subnet OCID. |
| `OCI_IMAGE_ID` | Ubuntu ARM image OCID for the selected region. |
| `OCI_AVAILABILITY_DOMAIN` | Availability domain, for example `nHas:AP-CHUNCHEON-1-AD-1`. |
| `OCI_SSH_PUBLIC_KEY` | One-line SSH public key, such as `ssh-ed25519 ...` or `ssh-rsa ...`. |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token. |
| `TELEGRAM_CHAT_ID` | Telegram chat ID that receives notifications. |

## Notes

- `OCI_CLI_KEY_CONTENT` must preserve all line breaks.
- Do not store any of these values in repository files.
- If a private key or Telegram token is exposed, rotate it before running the workflow.

