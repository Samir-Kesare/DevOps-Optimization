---
- name: Check password expiry for a specific user
  hosts: webservers
  gather_facts: yes
  vars_files:
    - vars.yaml

  tasks:
    - name: Get password expiry date for user {{ user_to_check }}
      shell: "chage -l {{ user_to_check }} | awk -F': ' '/Password expires/ {print $2}'"
      register: expiry_output
      changed_when: false

    - name: Calculate days to expiry
      set_fact:
        days_to_expiry: >-
          {{
            (
              (expiry_output.stdout | to_datetime('%b %d, %Y') | to_datetime('%s') | int) -
              (ansible_date_time.epoch | int)
            ) // 86400
          }}
      when: expiry_output.stdout is defined and expiry_output.stdout != 'never'

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
