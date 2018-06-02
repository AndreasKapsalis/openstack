# openstack
Openstack installation notes for kolla

## Neutron
For the LBaaS to work properly edit the `neutron_lbaas.conf` and change  the `auth_url` and `auth_version` to point to version 3

## Magnum
### Cinder
Magnum requires a volume type for attaching volume to clusters. Create a volume type:
```
openstack volume type create volType1 --description "Fix for Magnum" --public
```
Add the following to `magnum.conf`
```
[cinder]
default_docker_volume_type = volType1
```
### Kubernetes
Server certificates are not generated due to error in the relevant script `make-cert.sh`. The script creates a concatenated string of the VMs IPs. For the public IP the script tries to retrieve it from the metadata API. If `--floating-ip-disabled` is chosen during cluster template creation, the metadata server returns an empty string. This causes the openssl command to return an error. To fix that issue edit the `/var/lib/kolla/venv/lib/python2.7/site-packages/magnum/drivers/common/templates/kubernetes/fragments/make-cert.sh` for the magnum_api and magnum_conductor containers. In line 40, remove the `$KUBE_NODE_PUBLIC_IP` parameter or replace with the private ip.

Due to this bug (https://bugs.launchpad.net/magnum/+bug/1751409) Magnum is not able to generate the CA key. A patch for `/var/lib/kolla/venv/lib/python2.7/site-packages/magnum/drivers/common/templates/kubernetes/fragments/configure-kubernetes-master.sh` is avaiable at https://github.com/openstack/magnum/commit/5a34d7d830ad4b6a714f079d4575e1705df434f3#diff-5149fbecc3eedea0e14cb30ed34b82b1. Change line `62` with the following:
```
-if [ -n "$CERT_MANAGER_API" ]; then
+if [ "$(echo $CERT_MANAGER_API | tr '[:upper:]' '[:lower:]')" = "true" ]; then
     KUBE_CONTROLLER_MANAGER_ARGS="$KUBE_CONTROLLER_MANAGER_ARGS --cluster-signing-cert-file=$CERT_DIR/ca.crt --cluster-signing-key-file=$CERT_DIR/ca.key"
 fi
 ```

### Swarm
Server certificates are not generated due to error in the relevant script `make-cert.sh`. The script creates a concatenated string of the VMs IPs. For the public IP the script tries to retrieve it from the metadata API. If `--floating-ip-disabled` is chosen during cluster template creation, the metadata server returns an empty string. This causes the openssl command to return an error. To fix that issue edit the `/var/lib/kolla/venv/lib/python2.7/site-packages/magnum/drivers/common/templates/swarm/fragments/make-cert.py` for the magnum_api and magnum_conductor containers. In line 76, remove the `_get_public_ip()` function call or replace with the private ip.
