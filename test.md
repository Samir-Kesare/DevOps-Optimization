---
- name: Check password expiry for a specific user
  hosts: webservers
  gather_facts: yes
  vars_files:
    - vars.yaml

  tasks:
    - name: Check if user {{ user_to_check }} exists
      shell: id -u {{ user_to_check }}
      register: user_check
      failed_when: false
      changed_when: false

    - name: Skip if user does not exist
      debug:
        msg: "User {{ user_to_check }} does not exist on {{ inventory_hostname }}"
      when: user_check.rc != 0

    - name: Get password expiry date for user {{ user_to_check }}
      shell: "chage -l {{ user_to_check }} | awk -F': ' '/Password expires/ {print $2}'"
      register: expiry_output
      changed_when: false
      when: user_check.rc == 0

    - name: Skip if password never expires
      debug:
        msg: "User {{ user_to_check }} on {{ inventory_hostname }} has no expiry (never)"
      when: expiry_output.stdout is defined and expiry_output.stdout == 'never'

    - name: Parse expiry date into datetime object
      set_fact:
        expiry_date_obj: "{{ expiry_output.stdout | to_datetime('%b %d, %Y') }}"
      when:
        - expiry_output.stdout is defined
        - expiry_output.stdout != 'never'

    - name: Parse current date into datetime object
      set_fact:
        current_date_obj: "{{ ansible_date_time.date + ' ' + ansible_date_time.time | to_datetime('%Y-%m-%d %H:%M:%S') }}"
      when: expiry_output.stdout != 'never'

    - name: Calculate number of days to expiry
      set_fact:
        days_to_expiry: "{{ ((expiry_date_obj - current_date_obj).total_seconds() // 86400) | int }}"
      when:
        - expiry_date_obj is defined
        - current_date_obj is defined

    - name: Show warning for expiring passwords
      debug:
        msg: >-
          {{ inventory_hostname }}: password for {{ user_to_check }} expires in {{ days_to_expiry }} days 🚨 Warning: Expiry within {{ warning_days }} days!
      when: days_to_expiry is defined and (days_to_expiry | int) <= (warning_days | int)
      register: warning_msg

    - name: Append warning to summary file
      lineinfile:
        path: /var/lib/jenkins/ansible/warning_summary.txt
        line: |
          ok: [{{ inventory_hostname }}] => {
              "msg": "{{ warning_msg.msg }}"
          }
        create: yes
        mode: '0644'
        state: present
        insertafter: EOF
      when: warning_msg is defined and warning_msg.msg is defined
      delegate_to: localhost

