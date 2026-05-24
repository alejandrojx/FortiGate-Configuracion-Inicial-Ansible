# Security Notes

This repository is designed to be public and does not include real FortiGate credentials, customer data, private IP addressing, or production configuration exports.

Before using it in a real environment:

- Store credentials in environment variables, Ansible Vault, or a secure CI/CD secret store.
- Do not commit `backups/`, `.env`, `vault.yml`, or exported FortiGate configuration files.
- Use least-privilege FortiGate admin accounts or API tokens.
- Test playbooks in a lab VDOM before applying them to production.
- Review `ansible_httpapi_validate_certs`. Certificate validation is disabled in the lab inventory for convenience.

