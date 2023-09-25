debops-gpgkey
=============

Wrap up `juju4.gpgkey_generate` to ease integration with DebOps.

This role takes the multi layered variables used by DebOps and processes them down to pass in to `gpgkey_generate` so it can create the keys.

Known problems
--------------

* Requires three custom branches present in https://github.com/medeopolis/ansible-gpgkey_generate (they are currently being upstreamed)
* Doesn't support generating multiple keys in one pass (not currently required for our usecase and sacrificed for expedience)


Role Variables
--------------

```
# Which user will be used while generating the key
gpgkey__generate_as: 'root'
# Where `.gnupg` will be created; could be home directory of the user
# generating the key
gpgkey__generate_home: "~{{ gpgkey__generate_as }}/"
# Notionally user generating key; prefixed on key filenames during export
gpgkey__export_prefix: "{{ gpgkey__generate_as }}"
# String ("Real name") to associate with key
gpgkey__generate_realname: "GPG key generated via Ansible"
# Email address to associate with key
gpgkey__generate_email: "{{ gpgkey__generate_as }}@{{ inventory_hostname }}"
# File to store passphrase in
gpgkey__generate_passphrase_path: "{{ secret + '/credentials/' + inventory_hostname + '/gpg/' }}"
gpgkey__generate_passphrase_file: 'passphrase.key'
# Passphrase to use. Created if it isn't set
gpgkey__generate_passphrase: "{{ lookup('password', gpgkey__generate_passphrase_path + gpgkey__generate_passphrase_file ) }}"

# Default key lifetime
gpgkey__generate_expiry: '5y'
# Pull exports of generated keys back to the controller
gpgkey__retrieve_keys: true
gpgkey__retrieve_target: "{{ secret + '/gpg/by-host/' + inventory_hostname }}"
# Filenames for exported GPG keys
gpgkey__export_public_key_name: "{{ gpgkey__generate_realname }}-export.pub"
gpgkey__export_private_key_name: "{{ gpgkey__generate_realname }}-export.priv"
gpgkey__export_fingerprint_name: "{{ gpgkey__generate_realname }}.finterprint"
```


Dependencies
------------

- Requires juju4.gpgkey_generate (which does all the work)
- Requires DebOps to be in use, core and PKI


Example Playbook
----------------

Playbook including two essential debops roles (core and PKI). Some vars are
included for illustrative purposes.

    - name: Generate GPG keys using defaults
      hosts: debops_service_duply
      become: true
      become_user: root
      vars:
        gpgkey__generate_home: "{{ pki_root }}/gpg/"
        gpgkey__generate_realname: "Generated for backups"
        gpgkey__generate_email: "noreply+{{ inventory_hostname_short }}@example.com"
      roles:
        - role: debops.debops.core
        - role: debops.debops.pki
        - role: medeopolis.debops-gpgkey


License
-------

GPLv3

