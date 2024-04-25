stuttgart-things/create-os-user
===============================

manage users and groups on linux systems.

<details><summary>VARIABLES</summary>

Add a users variable containing the list of users to add. A good place to put
this is in `group_vars/all` or `group_vars/groupname` if you only want the
users to be on certain machines.

The following attributes are required for each user:

* `username` - The user's username.
* `name` - The full name of the user (gecos field).
* `home` - The home directory of the user to create (optional, defaults to /home/username).
* `uid` - The numeric user id for the user (optional). This is required for uid consistency
  across systems.
* `gid` - The numeric group id for the group (optional). Otherwise, the
  `uid` will be used.
* `password` - If a hash is provided then that will be used, but otherwise the
  account will be locked.
* `update_password` - This can be either 'always' or 'on_create'
  - `'always'` will update passwords if they differ. (default)
  - `'on_create'` will only set the password for newly created users.
* `group` - Optional primary group override.
* `groups` - A list of supplementary groups for the user.
* `append` - If yes, will only add groups, not set them to just the list in groups (optional).
* `profile` - A string block for setting custom shell profiles.
* `ssh_key` - This should be a list of SSH keys for the user (optional). Each SSH key
  should be included directly and should have no newlines.
* `generate_ssh_key` - Whether to generate a SSH key for the user (optional, defaults to no).
* `enable_ssh_tcp_forwarding` - Whether to set SSH-TCP Forwording or not
 
In addition, the following items are optional for each user:

* `shell` - The user's shell. This defaults to /bin/bash. The default is
  configurable using the users_default_shell variable if you want to give all
  users the same shell, but it is different than /bin/bash.
* `is_system_user` -  Set to `True` to create system user.

</details>

<details><summary>DEFAULTS</summary>

* `users_create_per_user_group` (default: true) - when creating users, also
  create a group with the same username and make that the user's primary
  group.
* `users_group` (default: users) - if users_create_per_user_group is _not_ set,
  then this is the primary group for all created users.
* `users_default_shell` (default: /bin/bash) - the default shell if none is
  specified for the user.
* `users_create_homedirs` (default: true) - create home directories for new
  users. Set this to false if you manage home directories separately.
* `deletion_user_group` (default: false) - delete group(s) and user(s). Set this to true to only
  run delete tasks.

</details>

<details><summary>ROLE INSTALLATION</summary>

installs role and all of it's dependencies w/:

```bash
cat <<EOF > /tmp/requirements.yaml
roles:
- src: https://github.com/stuttgart-things/create-os-user.git
  scm: git
collections:
- name: ansible.posix
EOF

ansible-galaxy install -r /tmp/requirements.yaml --force
ansible-galaxy collection install -r /tmp/requirements.yaml --force
rm -rf /tmp/requirements.yaml
```

</details>

<details><summary>EXAMPLE INVENTORY</summary>

```bash
cat <<EOF > inventory
[appserver]
1.2.3.4 ansible_user=sthings
EOF
```

</details>

<details><summary>EXAMPLE PLAYBOOK - USERS DEFINITION INLINE</summary>

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
cat <<EOF > create-os-user.yaml
- hosts: "{{ target_host }}"
  gather_facts: true
  become: true
  roles:
    - role: create-os-user
      vars:
        users:
          - username: rke
            name: rke user
            groups: ['{{ admin_group }}', 'k8s-admins']
            uid: 1005
            home: /home/rke
            profile: |
              alias ll='ls -ahl'
            generate_ssh_key: true
            #ssh_key:
            #  - "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
            enable_ssh_tcp_forwarding: true

        groups_to_create:
          - name: k8s-admins
            gid: 11000
          - name: developers
            gid: 21000

        users_deleted:
          - username: rke
            uid: 1005
            home: /home/rke/
            remove: true
            force: true

        groups_deleted:
          - name: developers
            gid: 21000
EOF
```

</details>

<details><summary>EXAMPLE PLAYBOOK - USERS DEFINITION VIA VARS FILE</summary>

You can optionally choose to remove the user's home directory and mail spool with

```yaml
cat <<EOF > users.yaml
---
users:
  - username: rke
    name: rke user
    groups: ['{{ admin_group }}', 'k8s-admins']
    uid: 1005
    home: /home/rke
    profile: |
      alias ll='ls -ahl'
    generate_ssh_key: true
    #ssh_key:
    #  - "{{ lookup('file', '/home/rke/.ssh/id_rsa.pub') }}"
    enable_ssh_tcp_forwarding: true
  - username: test1
    name: test1 user
    groups: ['{{ admin_group }}', 'k8s-admins']
    uid: 1006
    home: /home/test1
    profile: |
      alias ll='ls -ahl'
    generate_ssh_key: true
    #ssh_key:
    #  - "{{ lookup('file', '/home/test1/.ssh/id_rsa.pub') }}"
    enable_ssh_tcp_forwarding: true
  - username: test2
    name: test2 user
    groups: ['{{ admin_group }}', 'developers']
    uid: 1007
    home: /home/test2
    profile: |
      alias ll='ls -ahl'
    generate_ssh_key: true
    #ssh_key:
    #  - "{{ lookup('file', '/home/test2/.ssh/id_rsa.pub') }}"
    enable_ssh_tcp_forwarding: true

groups_to_create:
  - name: k8s-admins
    gid: 11000
  - name: developers
    gid: 21000

users_deleted:
  - username: rke
    uid: 1005
    home: /home/rke/
    remove: true
    force: true
  - username: test1
    uid: 1006
    home: /home/test1/
    remove: true
    force: true
  - username: test2
    uid: 1007
    home: /home/test2/
    remove: true
    force: true

groups_deleted:
  - name: k8s-admins
    gid: 11000
  - name: developers
    gid: 21000
EOF
```

```yaml
cat <<EOF > create-os-user.yaml
---
- hosts: "{{ target_host }}"
  gather_facts: true
  become: true
  vars_files:
    - users.yaml

  roles:
    - role: create-os-user
EOF
```

</details>

<details><summary>EXAMPLE PLAYBOOK - COPY PUBLIC (SSH-)KEY TO AUTHORIZED_KEYS FILE</summary>

```yaml
cat <<EOF > users.yaml
---
users:
  - username: rke
    name: rke user
    groups: ['{{ admin_group }}', 'k8s-admins']
    uid: 1005
    home: /home/rke
    profile: |
      alias ll='ls -ahl'
    generate_ssh_key: false # Default is true
    ssh_key: # Commented in
      - "{{ lookup('file', '/home/rke/.ssh/id_rsa.pub') }}"
    enable_ssh_tcp_forwarding: true

groups_to_create:
  - name: k8s-admins
    gid: 11000
  - name: developers
    gid: 21000
EOF
```

```yaml
cat <<EOF > create-os-user.yaml
---
- hosts: "{{ target_host }}"
  gather_facts: true
  become: true
  vars_files:
    - users.yaml

  roles:
    - role: create-os-user
EOF
```

</details>

<details><summary>EXAMPLE CONFIG - REMOVE USERS</summary>

```yaml
cat <<EOF > users.yaml
---
users_deleted:
  - username: rke
    uid: 1005
    home: /home/rke/
    remove: true
    force: true

groups_deleted:
  - name: k8s-admins
    gid: 11000
  - name: developers
    gid: 21000
EOF
```

</details>

<details><summary>EXAMPLE GENERATING HASHED PASSWORD</summary>

```bash
ansible all -i localhost, -m debug -a "msg={{ 'password' | password_hash('sha512', 'mysecretsalt') }}"
```
</details>

<details><summary>EXAMPLE EXECUTION</summary>

```bash
# Command to create group(s) and user(s)
ansible-playbook -i inventory create-os-user.yaml -vv

# Command to delete group(s) and user(s)
ansible-playbook -i inventory create-os-user.yaml -vv --extra-vars "deletion_user_group=true"
```

</details>

## License
<details><summary>LICENSE</summary>

Copyright 2020 patrick hermann.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
</details>

Role history
----------------
| date  | who | changelog |
|---|---|---|
|2024-04-22   | Andre Ebert | Added linting and task to prompt for user password and to delete user directories 
|2020-10-10   | Patrick Hermann | Updated for using of ansible collections, fixed role structure, default 'admin-user group' for debian/redhat
|2020-04-20   | Patrick Hermann | intial commit for creation of users/groups w/ ansible

Author Information
------------------

```yaml
Andre Ebert, 04/2024, andre.ebert@sva.de, Stuttgart-Things
Patrick Hermann, 03/2020, patrick.hermann@sva.de, Stuttgart-Things
```
