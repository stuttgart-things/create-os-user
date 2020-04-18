create-os-user
=========

Role to manage users on a system/os.

Requirements
------------

none.

Defaults
--------------

* `users_create_per_user_group` (default: true) - when creating users, also
  create a group with the same username and make that the user's primary
  group.
* `users_group` (default: users) - if users_create_per_user_group is _not_ set,
  then this is the primary group for all created users.
* `users_default_shell` (default: /bin/bash) - the default shell if none is
  specified for the user.
* `users_create_homedirs` (default: true) - create home directories for new
  users. Set this to false if you manage home directories separately.

user vars
------------

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

Example vars Adding users
-------------------------
```
    ---
    vars:
      users:
        - username: rke 
          name: rke user
          groups: ['wheel','docker']
          uid: 1005
          home: /home/rke
          profile: |
            alias ll='ls -ahl'
          ssh_key:
            - "{{ lookup('file', '/root/.ssh/rancher_rsa.pub') }}"
          enable_ssh_tcp_forwarding: True
        
      groups_to_create:
        - name: developers
          gid: 20000
```

Example generating hashed password
----------------------------------

```
ansible all -i localhost, -m debug -a "msg={{ 'password' | password_hash('sha512', 'mysecretsalt') }}
```


Example vars Removing users
---------------------------

You can optionally choose to remove the user's home directory and mail spool with
the `remove` parameter, and force removal of files with the `force` parameter.

    users_deleted:
      - username: vagrant
        uid: 1003
        remove: yes
        force: yes


Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```
- hosts: all
  remote_user: root
  gather_facts: true
  roles:
    - role: create-os-user 
      vars:
        users:
          - username: rke 
            name: rke user
            groups: ['wheel','docker']
            uid: 1005
            home: /home/rke
            profile: |
              alias ll='ls -ahl'
            ssh_key:
              - "{{ lookup('file', '/root/.ssh/rancher_rsa.pub') }}"
            enable_ssh_tcp_forwarding: True
        
        groups_to_create:
          - name: developers
            gid: 20000

      roles:
         - role: create-os-user
```
License
-------

BSD

Author Information
------------------

Patrick Hermann, 04/2020
patrick.hermann@sva.de