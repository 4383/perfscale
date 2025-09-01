## PTG
- performance / scale testing: https://etherpad.opendev.org/p/oct2024-ptg-neutron#L349
- Defining threshold: https://github.com/openstack/neutron/blob/53db0e2c6dbcc99f862c2f91457ecf8d69f20326/rally-jobs/task-neutron.yaml#L34

## devstack
- https://review.opendev.org/c/openstack/devstack/+/932203
- https://review.opendev.org/c/openstack/devstack/+/932199

## neutron
- https://review.opendev.org/c/openstack/neutron/+/941906
- https://review.opendev.org/c/openstack/neutron/+/941531
- https://review.opendev.org/c/openstack/neutron/+/941211
- https://review.opendev.org/c/openstack/neutron/+/941092
- https://review.opendev.org/c/openstack/neutron/+/938393
- https://review.opendev.org/c/openstack/neutron/+/939315
- https://review.opendev.org/c/openstack/neutron/+/935668
- https://review.opendev.org/c/openstack/neutron/+/935626
- https://review.opendev.org/c/openstack/neutron/+/931012
- https://review.opendev.org/c/openstack/neutron/+/932189
- https://review.opendev.org/c/openstack/neutron/+/932592
- https://review.opendev.org/c/openstack/neutron/+/930889
- https://review.opendev.org/c/openstack/neutron/+/924317
- https://review.opendev.org/c/openstack/neutron/+/923711

## Commits
- cd3957a0f9 Move grenade skip level jobs to wsgi
- 86f94de99a [eventlet-removal] Remove logger mechanism from remaining ML2/OVN CI jobs
- 57e1f20373 [eventlet-removal] Explicitly define the mechanism driver
- ceafa66231 [eventlet-removal] Remove "logger" mechanism from ML2/OVN CI jobs
- 0b975a679f Switch dvr grenade job Neutron API WSGI module
- 1dabdf118d [eventlet-removal] Use non-eventlet metadata proxy in OVN metadata agent
- adba4b960c Move neutron rally jobs to wsgi
- dc21fdb8e5 Merge "Remove all eventlet Neutron API jobs"
- 09e13ffad0 Make grenade jobs work with Neutron API WSGI module
- 0415d3a41a Remove all eventlet Neutron API jobs
- bd82a35b33 Merge "Add Neutron API eventlet experimental jobs"
- 34b0ce6aea Add Neutron API eventlet experimental jobs
- 7fe20024ef zuul: Explicitly set NEUTRON_DEPLOY_MOD_WSGI
- 3552c334c0 [WSGI] Move all OVS jobs to use WSGI API module (2)
- 16fc47127a Disable the OVN/OVS WGSI experimental jobs
- deb99de993 [WSGI] Move all OVN jobs to use WSGI API module
- 19208ebdae [WSGI] Move all OVS jobs to use WSGI API module
