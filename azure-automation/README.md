# Azure SIG Image Management

This repository contains an Ansible playbook and role to create Azure Shared Image Gallery (SIG) image definitions and image versions from an existing Virtual Machine.

Files
- `azure-sig-image-management.yml` - main playbook that includes the `azure_sig` role.
- `roles/azure_sig/tasks/main.yml` - task list that implements the SIG creation workflow.

Overview

The playbook collects information about a source VM, creates (or ensures) a gallery image definition, optionally determines or bumps the gallery image version, and creates a gallery image version from the VM. It will stop the VM before image capture (if it was running) and start it again afterwards.

Prerequisites

- Ansible (2.9+ recommended).
- The `azure.azcollection` Ansible collection. Install with:

```bash
ansible-galaxy collection install azure.azcollection
```

- Python pip packages (install in the same Python environment Ansible uses):

```bash
pip install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements.txt
```

- Azure credentials available to Ansible. Common options:
	- Sign in with `az login` (Azure CLI) and enable the CLI-based auth flow, or
	- Export a Service Principal environment variables (`AZURE_CLIENT_ID`, `AZURE_SECRET`, `AZURE_SUBSCRIPTION_ID`, `AZURE_TENANT`), or
	- Configure a credential plugin supported by `azure.azcollection`.

Inventory

This playbook uses `connection: local` and runs against the control host. Your inventory can be as simple as `localhost` or any host group. Example run uses the `all` hosts group configured for local execution.

Variables

The role expects the following variables to be supplied (via `-e`, `group_vars`, or another inventory mechanism):

- `vm_resource_group` (required): Resource group of the source VM.
- `vm_name` (required): Name of the source virtual machine to capture.
- `gallery_resource_group` (required): Resource group where the Shared Image Gallery lives (or will be created/used).
- `gallery_name` (required): Name of the Shared Image Gallery.
- `offer_name` (required): Publisher/offer identifier used in the image identifier (example: `EXAMPLE` in the role — replace as appropriate).
- `sku_version` (required): SKU string to include in the image identifier (e.g., `1.0.0` or semantic SKU).
- `version` (optional): Specific gallery image version to create (e.g., `1.0.1`). If omitted, the role will fetch existing versions and auto-bump the patch portion of the latest version.

Notes on behavior

- If `version` is not provided, the role fetches SIG versions, selects the latest, increments the last numeric segment, and uses that as the new version.
- The role will stop the VM prior to creating a gallery image version if it was running; it will restart the VM afterwards regardless of the snapshot result.
- If no existing SIG versions are found and `version` is not provided, the playbook will fail and prompt you to supply a `version`.

Example usage

Run the playbook by passing required variables on the command line:

```bash
ansible-playbook azure-sig-image-management.yml \
	-e "vm_resource_group=my-vm-rg vm_name=my-vm gallery_resource_group=my-gallery-rg gallery_name=myGallery offer_name=myPublisher sku_version=1.0.0"
```

To specify an explicit version instead of auto-bumping:

```bash
ansible-playbook azure-sig-image-management.yml \
	-e "vm_resource_group=my-vm-rg vm_name=my-vm gallery_resource_group=my-gallery-rg gallery_name=myGallery offer_name=myPublisher sku_version=1.0.0 version=1.0.5"
```

Tips and Troubleshooting

- If you see authentication or permission errors, verify your Azure credentials and that the identity has `Contributor` or appropriate permissions on the target resource groups.
