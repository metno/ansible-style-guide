# MET Ansible Style Guide v1.0.0

## Index

* [Best practices](#best-practices)
* [The beginning of a file](#the-beginning-of-a-file)
* [The end of a file](#the-end-of-the-file)
* [Boolean variables](#boolean-variables)
* [Key/value pairs](#keyvalue-pairs)
* [Order in playbook](#order-in-playbook)
* [Order in task declaration](#order-in-task-declaration)
* [Air and tabulators](#air-and-tabulators)
* [Variable names](#variable-names)

## Best practices

Follow Ansible [Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html) and Ansible [Default Rules](https://docs.ansible.com/ansible-lint/rules/default_rules.html) for linting. You can see exceptions to lint rules for the current project in the file `.ansible-lint`.

### Why do this?

The guys and girls who created Ansible have a good understanding how playbooks work and where files should reside. You'll avoid a lot of your own ingenious pitfalls following their best practices.

### So why doesn't my code look like Ansible examples?

Examples in Ansible documentation are inconsistent. The main reason for this style guide is to have one consistent way of writing Ansible which is easy to read for you and others.

## The beginning of a file

Always start your file with the YAML heading which is `---`. Please add comments above these lines with descriptions of what this playbook is and an example of how to run it.

#### Don't do this

```yaml
- name: Change status of user foo
  service:
    enabled: true
    name: foo
    state: '{{ state }}'
  become: true
```

#### Do this

```yaml
# This playbook playbook changes state of user foo
# Example: ansible-playbook -e state=stopped playbook.yml
---
- name: Change status of user foo
  service:
    enabled: true
    name: foo
    state: '{{ state }}'
  become: true
```

### Why do this?

This makes it quick to find out what the playbook does. Either with opening the file or just using the `head` command.

## The end of the file

Always end the file with a line shift.

### Why do this?

It's just Unix best practices. It avoids messing up your prompt when you `cat` a file.

## Boolean variables

#### Don't do this

```yaml
---
- name: start chrony NTP daemon
  service:
    name: chrony
    state: started
    enabled: 1
  become: yes
```

#### Do this

```yaml
---
- name: start chrony NTP daemon
  service:
    name: chrony
    state: started
    enabled: true
  become: true
```

### Why do this?

Boolean variables may be expressed in a myriad of ways. `0/1`, `False/True`, `no/yes` and `false/true`. Even if you have a choice it's good to have a standard: `false/true`. There are a lot of scripting languages who have standardized on the `false/true` variant. Keep to it.


## Key/value pairs

Only use on space after colon when you define key value pair.

#### Don't do this

```yaml
---
- name: start chrony NTP daemon
  service:
    name    : chrony
    state   : started
    enabled : true
  become : true
```

#### Do this

```yaml
---
- name: start chrony NTP daemon
  service:
    name: chrony
    state: started
    enabled: true
  become: true
```

Keep to only one standard. In our case **always use the "map" syntax**. This is regardless of how many key/value pairs that exist in a map.

#### Don't do this

```yaml
---
- name: disable ntpd
  service: name='{{ ntp_service }}' state=stopped enabled=false
  become: true
```

#### Do this

```yaml
---
- name: disable ntpd
  service:
    name: '{{ ntp_service }}'
    state: stopped
    enabled: false
  become: true

```

### Why do this?

It's **soooo** much easier to read, and not more work to do. As the writer of this document is dyslectic, think of him and others in the same situation. In addition to the readability, it decreases the chance for a merge conflict.

## Order in playbook

Playbook definitions should follow this order.

* `name`
* `hosts`
* `remote_user`
* `become`
* `vars`
* `pre_tasks`
* `roles`
* `tasks`

```yaml
---
- name: update root authorized_keys for all machines
  hosts:
    - all
  become: yes

  vars:
    root_keys_users:
      - user: user1
        key: |
          ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHkCVF05JvfkrfOOESivOxV4N8+A/EMEkF7/nCQMRoQg
        enabled: true

  pre_tasks:
    - name: add custom root users
      set_fact:
        root_keys_users: "{{ root_keys_users | union( root_keys_users_custom|default([]) ) }}"

  roles:
    - root-keys

  tasks:
    - name: print out debug message
      debug:
        msg: "fee foo faa"
```

### Why do this?

A common order makes playbooks consistent and easier to read for your dear colleagues. Think of them when you write.

## Order in task declaration

A task should be declared in this order.

* `name`
* module and the arguments for the module
* arguments for the task in alphabetic order
* `tags` at the bottom

```yaml
---
- name: unhold packages
  shell: 'apt-mark -s unhold {{ item }} && apt-mark unhold {{ item }}'
  args:
    executable: /bin/bash
  changed_when: apt_mark_unhold.rc == 0 and "already" not in apt_mark_unhold.stdout
  failed_when: false
  loop: '{{ packages_ubuntu_unhold }}'
  register: apt_mark_unhold
  tags:
    - apt_unhold
---
```

### Why do this?

Reason is the same as "order of playbook". To make tasks more consistent and easier to read. Help your colleagues.

## Air and tabulators

Air, one of the **most important thing** for humans and **for code**! It must be an empty line before `vars`, `pre_tasks`, `roles` and `tasks`, and before each task in the tasks definition. Tabulator stops must be set to two, `2`, spaces.

### Why do this?

This creates a pretty and tidy code which is easy to read, both for dyslectic and non dyslectic people. There is no excuse not to do this.

## Variable names

Always use `snake_case` for variable names.

#### Don't do this

```yaml
---
- name: set my variables
  set_fact:
    aBoolean: false
    anint: 101
    A_STRING: bar
```

#### Do this

```yaml
---
- name: set my variables
  set_fact:
    a_boolean: false
    an_int: 101
    a_string: bar
```

### Why do this?

Ansible already uses `snake_case` for variables in it's examples. Consistent naming of variables keeps the code tidy and gives better readability.
