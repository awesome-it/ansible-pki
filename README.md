# Ansible PKI

Create self-signed PKI authorities and certificates using Cloudflare's PKI and TLS toolkit CFSSL.

## Cloudflare PKI and TLS toolkit: CFSSL

The playbooks require Cloudflare's PKI and TLS toolkit CFSSL from https://github.com/cloudflare/cfssl.
You can run the following to install CFSSL **locally**:

```shell
ansible-playbook install-cfssl.yml
```

## Howto create Certificate Authority & Certificates

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

## Howto to use in an existing Ansible Repository

Assuming the following setup:

* Your Ansible repo is in `~/myrepo`
* You have a group `dev` with hosts that require the PKI
* You want to create your PKI (including CA and certs) to `~/myrepo/group_vars/dev/pki`

You can do the following:

* Add the PKI repo as a submodule i.e. to `~/myrepo/shared/pki`:
    ```shell
    cd ~/myrepo
    git submodule add https://github.com/awesome-it/ansible-pki.git shared/pki
    ```
* Add the PKI config to `group_vars/dev/pki.yml`:
    ```shell
    cp ~/myrepo/shared/pki/group_vars.example.yml ~/myrepo/group_vars/dev/pki.yml
    ```
* Edit and adopt`group_vars/dev/pki.yml` to your needs.
* Copy the example playbooks to your Ansible Repo and replace `<group>` with `dev`:
    ```shell
    cp ~/myrepo/shared/pki/playbook-example/play-pki-*.yml ~/myrepo
    sed -i 's/<group>/dev/g' ~/myrepo/play-pki-*.yml
    ```
* Set the `out_dir` in the playbooks to `group_vars/dev/pki.yml`:
    ```yaml
    ---
    pki:
      out_dir: "{{ inventory_dir }}/../group_vars/dev/pki"
      default: ...
    ```
* Run the playbooks as described above:
  * `ansible-playbook play-pki-install-cfssl.yml`: Install CFSSL tools to `~/.local/bin`. 
  * `ansible-playbook play-pki-decrypt.yml`: Decrypt a potentially existing encrypted PKI.
  * `ansible-playbook play-pki-create-ca.yml`: Create the configured authorities.
  * `ansible-playbook play-pki-create-certs.yml`: Create the configured certificates.
    * Note that this won't overwrite existing certificates. You need to manually delete a certificate in order to update
      if i.e. to replace expired certificates.
  * `ansible-playbook play-pki-encrypt.yml`: Encrypt the generated PKI files.