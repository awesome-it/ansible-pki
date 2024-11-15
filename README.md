# Ansible PKI

Create self-signed PKI authorities and certificates using Cloudflare's PKI and TLS toolkit CFSSL.

## Cloudflare PKI and TLS toolkit: CFSSL

The playbooks require Cloudflare's PKI and TLS toolkit CFSSL from https://github.com/cloudflare/cfssl.
You can run the following to install CFSSL **locally**:

```shell
ansible-playbook install-cfssl.yml
```

## Certificate Authority & Certificates

The configuration for certificate for authorities and certificates can be defined in `pki.authorities` and `pki.certificates`
as described in the defaults in `roles/generate/defaults/main.yml`. 

To maintain multiple authorities, you can create multiple config files in `group_vars` and run the playbooks with the 
corresponding inventory file. See `group_vars` and the corresponding `inventory` files for examples. You can also find 
an example config in `group_vars.example.yml`.

All playbooks will **run locally** and will generate all files of the configured authority in `{{ out_dir }}/group`.

The `encrypt.yml` playbook will encrypt the generated PKI files from `{{ out_dir }}/<group>` into `{{ out_dir }}/group.tar.gz.enc`, 
so that it can be added to the Git repo using Git LFS (see `.gitattributes`).

Similar you can use `decrypt.yml` to decrypt the files from `{{ out_dir }}/group.tar.gz.enc` into `{{ out_dir }}/group` 
whenever you start working on the PKI again.

* Decrypt (potentially existing) PKI files before you start working on the PKI:
  ```shell
  ansible-playbook --inventory ./inventory/<inventory>.yml decrypt.yml
  ```

* Run `create-authority.yml` to create the configured certificate authorities:
  ```shell
  ansible-playbook --inventory ./inventory/<inventory>.yml create-authorities.yml 
  ```

* Run `create-certificates.yml` to create the configured certificates signed by the defined CAs.
  ```shell
  ansible-playbook --inventory ./inventory/<inventory>.yml create-certificates.yml 
  ```

* Encrypt generate PKI file in order to add them safely to the Git repo:
  ```shell
  ansible-playbook --inventory ./inventory/<inventory>.yml encrypt.yml
  ```

The playbooks `create-authorities.yml` and `create-certificates.yml` are idempotent and will only create files if they 
do not exist yet. This means that you have to manually delete (i.e. expired) certificates from `{{ out_dir }}/<group>` 
first, if you want to re-new them.

Further `decrypt.yml` and `encrypt.yml` won't overwrite existing files, so you have to manually delete them or set 
`force: true` in the playbook file.

If all else fails, you still have Git.