[jenkins@k8s-master devopscore_ansible]$ ansible-playbook -i inventory.ini check_expiry.yaml
/usr/lib/python2.7/site-packages/requests/__init__.py:91: RequestsDependencyWarning: urllib3 (1.24.1) or chardet (2.2.1) doesn't match a supported version!
  RequestsDependencyWarning)

PLAY [Check password expiry for a specific user] ************************************************************************************

TASK [Gathering Facts] **************************************************************************************************************
ok: [172.27.67.32]

TASK [Get password expiry date for user esbuser] ************************************************************************************
ok: [172.27.67.32]

TASK [Calculate days to expiry] *****************************************************************************************************
fatal: [172.27.67.32]: FAILED! => {"msg": "Invalid value for epoch value (%s)"}

PLAY RECAP **************************************************************************************************************************
172.27.67.32               : ok=2    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0

