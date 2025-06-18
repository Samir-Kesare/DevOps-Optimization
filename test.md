[jenkins@k8s-master devopscore_ansible]$ ansible-playbook -i inventory.ini check_expiry.yaml
/usr/lib/python2.7/site-packages/requests/__init__.py:91: RequestsDependencyWarning: urllib3 (1.24.1) or chardet (2.2.1) doesn't match a supported version!
  RequestsDependencyWarning)

PLAY [Check password expiry for a specific user] ************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************
ok: [172.27.67.32]

TASK [Get password expiry date for user esbuser] ************************************************************************************
ok: [172.27.67.32]

TASK [Calculate days to expiry] *****************************************************************************************************
fatal: [172.27.67.32]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'datetime.datetime object' has no attribute 'timestamp'\n\nThe error appears to be in '/app/jenkins/devops/devopscore_ansible/check_expiry.yaml': line 14, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n\n    - name: Calculate days to expiry\n      ^ here\n"}

PLAY RECAP **************************************************************************************************************************
172.27.67.32               : ok=2    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
