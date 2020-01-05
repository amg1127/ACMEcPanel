ACMEcPanel - Ansible role for ACME-automated certificate deployment on cPanel-managed websites
=========

This Ansible role allows automated installation of a certificate generated using ACME protocol into a website managed by cPanel. It uses [acme_certificate module](https://docs.ansible.com/ansible/latest/modules/acme_certificate_module.html) for certificate creation and renewal and [UAPI cPanel API](https://documentation.cpanel.net/display/DD/Guide+to+UAPI) for certificate deployment.

This role is useful for automated deployment of Let's Encrypt certificates.

Requirements
--------------

Depending on how cPanel is configured, this role may depend on `pexpect` and `sftp` binary in order to upload ACME challenge files successfully. Due to unknown reasons, I could not upload files to my production server (which is not under my control) using [Fileman::upload_files](https://documentation.cpanel.net/display/DD/UAPI+Functions+-+Fileman%3A%3Aupload_files) UAPI function and had to do so by calling `sftp` directly.


Role Variables
--------------

These are variables related to ACME instance (all values will be forwarded to `acme_certificate` module):
- `acme_account_email`: E-mail address associated with the ACME account.
- `acme_directory`: ACME endpoint URI.
- `acme_version`: ACME version of the endpoint.
- `acme_agreement`: URI to a terms of service document agreed to when using the ACME v1 service at `acme_directory`.
- `acme_terms_agreed`: Boolean indicating whether ACME service terms have been agreed.

These are variables related to cPanel instance:
- `cPanel_endpoint`: Hostname of the cPanel server. UAPI calls will be forwarded to `https://{{ cPanel_endpoint }}:2083/execute/`.
- `cPanel_user`: Username of a valid account for cPanel authentication.
- `cPanel_password`: Password of the account for cPanel authentication.
- `cPanel_websites`: An array containing domain names of all SSL websites managed by the cPanel server.

This role requires a permanent storage space for ACME account information, certificates and private keys. By default, data will be stored in `${XDG_DATA_HOME}/ACMEcPanel` (falling back to `${HOME}/.local/share/ACMEcPanel` if `${XDG_DATA_HOME}` is not defined). These are variables allows specifying alternative location for data storage and alternative filenames:
- `ACMEcPanel_storage_path`: Path that will be used for data storage.
- `ACMEcPanel_account_key_file`: Name of the file storing the ACME account key.
- `ACMEcPanel_certificate_key_file`: Name of the file storing the certificate key that will be uploaded to cPanel server.
- `ACMEcPanel_certificate_request_file`: Name of the file storing the certificate request that will be generated and submitted to the ACME endpoint.
- `ACMEcPanel_certificate_file`: Name of the file storing the certificate file that will be uploaded to cPanel server.
- `ACMEcPanel_certificate_chain_file`: Name of the file storing the certificate chain. It will be uploaded to cPanel server as CA bundle data.


Example Playbook
----------------

An example of how to use the role follows below:

    - hosts: localhost
      roles:
      - ACMEcPanel
      vars:
        acme_account_email: 'manager@example.com'
        acme_directory: 'https://acme-v02.api.letsencrypt.org/directory'
        acme_version: 2
        acme_terms_agreed: yes
        cPanel_endpoint: 'cpanel.example.com'
        cPanel_user: 'manager'
        cPanel_password: 'Pa$$w0rd'
        cPanel_websites:
        - 'example.com'
        - 'www.example.com'

The example above specifies `{{ cPanel_user }}` and `{{ cPanel_password }}` in cleartext within the playbook, which is not recommended. Using a vault file and loading its contents via command line is more secure:

```
$ ansible-vault create ./ACMEcPanel-vault.yml
$ ansible-playbook ACMEcPanel-playbook.yml --extra-vars @./ACMEcPanel-vault.yml --ask-vault-pass
```

License
-------

GPL3

Author Information
------------------

Anderson Medeiros Gomes <contact at andersongomes dot tech>

