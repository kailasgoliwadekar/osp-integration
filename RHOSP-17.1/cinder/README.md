
# Dell EMC Cinder Backend Deployment Guide for Red Hat OpenStack Platform 17

## Overview

This document describes how to deploy the Dell EMC Block Storage services in a Red Hat OpenStack Platform Overcloud.
This assumes that the RHOSP installation is done through RHOSP director toolset which is based primarily on the upstream TripleO project.  



The following Dell EMC storage drivers are fully integrated with director and can be deployed using tripleo heat templates 
* [XtremIO iSCSI and FC drivers](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/dell-emc-xtremio-driver.html) - see below
* [SC Series FC and iSCSI driver](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/dell-storagecenter-driver.html) - see below
* [Unity iSCSI and FC drivers](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/dell-emc-unity-driver.html) -  Please refer to this [custom deployment guide for the Unity Driver](https://github.com/emc-openstack/osp-deploy/tree/rhosp17.1/cinder)
* [VNX iSCSI and FC drivers](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/dell-emc-vnx-driver.html) - Please refer to this [custom deployment guide for the VNX driver](https://github.com/emc-openstack/osp-deploy/tree/rhosp16/cinder)
* [PowerFlex/VxFlex OS drivers](https://docs.openstack.org/cinder/train/configuration/block-storage/drivers/dell-emc-vxflex-driver.html) - Please refer to this [Guide for the PowerFlex/VXFlex OS driver](https://github.com/dell/osp-integration/tree/master/RHOSP-17.1/cinder/powerflex/README.md) 
* [PowerStore iSCSI and FC drivers](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/dell-emc-powerstore-driver.html) - see below
* [PowerMax iSCSI and FC drivers](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/dell-emc-powermax-driver.html) - see below

**Note:** The following drivers will be deprecated starting from the 2023.1 Antelope OpenStack upstream
* XtremIO iSCSI and FC drivers
* SC Series FC and iSCSI drivers
* VNX iSCSI and FC drivers

## Prerequisites
- Dell EMC Storage Backend configured as storage repository.
- Configuration settings and credentials.

## Deployment Steps

### Prepare the Environment File
The environment file is an OSP director environment file. The environment file contains the settings for each back end you want to define. Using the environment file will ensure that the back end settings persist through future Overcloud updates and upgrades.  

Create the environment file that will orchestrate the back end settings. Use the sample file provided below for your specific backend.  

**Note:** **LVM driver** is enabled by default in TripleO, you want to set the ```CinderEnableIscsiBackend``` to false in one of your environment file to turn it off.

```yaml
parameter_defaults:
  CinderEnableIscsiBackend: false
```

**1. XtremIO iSCSI and FC drivers**

For full detailed instruction of all options please refer to [XtremIO Backend Configuration](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/dell-emc-xtremio-driver.html#configuration-options).

**Environment sample**

With a director deployment, XtremIO backend can be deployed using the integrated heat environment file. This file is located in the following path of the Undercloud node:
/usr/share/openstack-tripleo-heat-templates/environments/cinder-dellemc-xtremio-config.yaml

Copy this file to a local path where you can edit and invoke it later. For example, to copy it to ~/templates/:

```bash
$ cp /usr/share/openstack-tripleo-heat-templates/environments/cinder-dellemc-xtremio-config.yaml ~/templates/
```
Afterwards, open the copy (~/templates/cinder-dellemc-xtremio-config.yaml) and edit it as you see fit. The following shows a sample content of the file. The files will list optional params that the user can choose to override if they don't like the default value.

```CinderXtremioStorageProtocol``` default value is iSCSI. To configure FC backend add ```CinderXtremioStorageProtocol: 'FC'```.

Note that the ```resource_registry``` entry in the heat environment file must be an absolute path when you make a copy.

```yaml
# A Heat environment file which can be used to enable a
# Cinder Dell EMC Xtremio backend, configured via puppet
resource_registry:
  OS::TripleO::Services::CinderBackendDellEMCXtremio: ../deployment/cinder/cinder-backend-dellemc-xtremio-puppet.yaml

parameter_defaults:
  CinderEnableXtremioBackend: true
  CinderXtremioBackendName: 'tripleo_dellemc_xtremio'
  CinderXtremioSanIp: '10.10.10.10'
  CinderXtremioSanLogin: 'my_username'
  CinderXtremioSanPassword: 'my_password'
  CinderXtremioClusterName: 'Cluster-Name'
  CinderXtremioArrayBusyRetryCount: 5
  CinderXtremioArrayBusyRetryInterval: 5
  CinderXtremioVolumesPerGlanceCache: 100
# To configure multiple backends, use CinderXtremioMultiConfig to
# assign parameter values specific to that backend. CinderXtremioMultiConfig
# is a dictionary of the parameter values for each backend specified in
# CinderXtremioBackendName.
# For example, see below:
#   CinderXtremioBackendName:
#     - tripleo_dellemc_xtremio_1
#     - tripleo_dellemc_xtremio_2
#   CinderXtremioMultiConfig:
#     tripleo_dellemc_xtremio_1:
#       CinderXtremioStorageProtocol: 'iSCSI' # Specific value for this backend
#     tripleo_dellemc_xtremio_2:
#       CinderXtremioStorageProtocol: 'FC' # Specific value for this backend
```

**2. PowerMax iSCSI and FC drivers**

For full detailed instruction of options please refer to [PowerMax Backend Configuration](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/dell-emc-powermax-driver.html#configuration-options)

**iSCSI Environment sample**

```yaml
parameter_defaults:
  ControllerExtraConfig:
    cinder::config::cinder_config:
      tripleo_dellemc_powermax/volume_driver:
        value: cinder.volume.drivers.dell_emc.powermax.iscsi.PowerMaxISCSIDriver
      tripleo_dellemc_powermax/volume_backend_name:
        value: tripleo_dellemc_powermax
      tripleo_dellemc_powermax/san_ip:
        value: '10.10.10.10'
      tripleo_dellemc_powermax/san_login:
        value: 'my_username'
      tripleo_dellemc_powermax/san_password:
        value: 'my_password'
      tripleo_dellemc_powermax/powermax_port_groups:
        value: '[OS-ISCSI-PG]'
      tripleo_dellemc_powermax/powermax_array:
        value: '000123456789'
      tripleo_dellemc_powermax/powermax_srp:
        value: 'SRP_1'
    cinder_user_enabled_backends: ['tripleo_dellemc_powermax']
```

**FC Environment sample**

```yaml
parameter_defaults:
  ControllerExtraConfig:
    cinder::config::cinder_config:
      tripleo_dellemc_powermax/volume_driver:
        value: cinder.volume.drivers.dell_emc.powermax.fc.PowerMaxFCDriver
      tripleo_dellemc_powermax/volume_backend_name:
        value: tripleo_dellemc_powermax
      tripleo_dellemc_powermax/san_ip:
        value: '10.10.10.10'
      tripleo_dellemc_powermax/san_login:
        value: 'my_username'
      tripleo_dellemc_powermax/san_password:
        value: 'my_password'
      tripleo_dellemc_powermax/powermax_port_groups:
        value: '[OS-FC-PG]'
      tripleo_dellemc_powermax/powermax_array:
        value: '000123456789'
      tripleo_dellemc_powermax/powermax_srp:
        value: 'SRP_1'
    cinder_user_enabled_backends: ['tripleo_dellemc_powermax']
```
**3. SC Series iSCSI and FC drivers**  

For full detailed instruction of options please refer to [SC Series Backend Configuration](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/dell-storagecenter-driver.html#configuration-options)

**iSCSI Environment sample**

With a director deployment, Dell SC Series backend can be deployed using the integrated heat environment file. This file is located in the following path of the Undercloud node:
/usr/share/openstack-tripleo-heat-templates/environments/cinder-dellsc-config.yaml

Copy this file to a local path where you can edit and invoke it later. For example, to copy it to ~/templates/:

```bash
$ cp /usr/share/openstack-tripleo-heat-templates/environments/cinder-dellemc-sc-config.yaml ~/templates/
```
Afterwards, open the copy (~/templates/cinder-dellemc-sc-config.yaml) and edit it as you see fit. The following shows a sample content of the file. The files will list optional params that the user can choose to override if they don't like the default value.

```yaml
# A Heat environment file which can be used to enable a
# Cinder Dell EMC Storage Center ISCSI backend, configured via puppet
resource_registry:
  OS::TripleO::Services::CinderBackendDellSc: ../deployment/cinder/cinder-backend-dellemc-sc-puppet.yaml

parameter_defaults:
  CinderEnableDellScBackend: true
  CinderDellScBackendName: 'tripleo_dellsc'
  CinderDellScSanIp: '10.10.10.10'
  CinderDellScSanLogin: 'my_username'
  CinderDellScSanPassword: 'my_password'
  CinderDellScSsn: 64702
  CinderDellScIscsiIpAddress: '10.10.10.11'
  CinderDellScServerFolder: 'dellsc_server'
  CinderDellScVolumeFolder: 'dellsc_volume'
  CinderDellScExcludedDomainIps: []
  CinderDellScMultipathXfer: true
```

**FC Environment sample**
``` yaml
parameter_defaults:
  ControllerExtraConfig:
    cinder::config::cinder_config:
      tripleo_dellemc_dellsc/volume_driver:
        value: cinder.volume.drivers.dell_emc.sc.storagecenter_fc.SCFCDriver
      tripleo_dellemc_dellsc/volume_backend_name:
        value: tripleo_dellemc_dellsc
      tripleo_dellemc_dellsc/san_ip:
        value: '10.10.10.1'
      tripleo_dellemc_dellsc/san_login:
        value: 'Admin'
      tripleo_dellemc_dellsc/san_password:
        value: 'my_password'
      tripleo_dellemc_dellsc/dell_sc_ssn :
        value: '64702'
      tripleo_dellemc_dellsc/dell_sc_server_folder:
        value: 'cindersrv'
      tripleo_dellemc_dellsc/dell_sc_volume_folder :
        value: 'cindervol'
    cinder_user_enabled_backends: ['tripleo_dellemc_dellsc']
```    
**4. PowerStore iSCSI and FC drivers**  
For full detailed instruction of options please refer to [PowerStore Backend Configuration](https://docs.openstack.org/cinder/latest/configuration/block-storage/drivers/dell-emc-powerstore-driver.html#driver-configuration)

**Environment sample**

With a director deployment, PowerStore backend can be deployed using the integrated heat environment file. This file is located in the following path of the Undercloud node:
/usr/share/openstack-tripleo-heat-templates/environments/cinder-dellemc-powerstore-config.yaml

Copy this file to a local path where you can edit and invoke it later. For example, to copy it to ~/templates/:

```bash
$ cp /usr/share/openstack-tripleo-heat-templates/environments/cinder-dellemc-powerstore-config.yaml ~/templates/
```
Afterwards, open the copy (~/templates/cinder-dellemc-powerstore-config.yaml) and edit it as you see fit. The following shows a sample content of the file. The files will list optional params that the user can choose to overwrite if they don't like the default value.

```yaml
# A Heat environment file which can be used to enable a
# Cinder Dell EMC PowerStore backend, configured via puppet.
resource_registry:
  OS::TripleO::Services::CinderBackendDellEMCPowerStore: ../deployment/cinder/cinder-backend-dellemc-powerstore-puppet.yaml

parameter_defaults:
  CinderEnablePowerStoreBackend: true
  CinderPowerStoreBackendName: 'tripleo_dellemc_powerstore'
  CinderPowerStoreMultiConfig: {}
  CinderPowerStoreAvailabilityZone: ''
  CinderPowerStoreSanIp: '10.10.10.10'
  CinderPowerStoreSanLogin: 'admin'
  CinderPowerStoreSanPassword: 'password'
  CinderPowerStoreAppliances: 'appliance-1,appliance2'
  CinderPowerStorePorts: '10.10.10.11,10.10.10.12'
  CinderPowerStoreStorageProtocol: 'iSCSI'

# To configure multiple PowerStore backends differently, use CinderPowerStoreMultiConfig to
# assign parameter values specific to each backend. For example:
#   CinderPowerStoreBackendName:
#     - tripleo_dellemc_powerstore_1
#     - tripleo_dellemc_powerstore_2
#   CinderPowerStoreMultiConfig:
#     tripleo_dellemc_powerstore_1:
#       CinderPowerStoreStorageProtocol: 'iSCSI' # Specific value for this backend
#     tripleo_dellemc_powerstore_2:
#       CinderPowerStoreStorageProtocol: 'FC' # Specific value for this backend
```    
### Deploy the configured backends

When you have created the file dellemc-backend-env.yaml file with appropriate backends, deploy the backend configuration by running the openstack overcloud deploy command using the templates option. If you passed any extra environment files when you created the overcloud, pass them again here using the -e option. 
 
```bash
(undercloud) $ openstack overcloud deploy --templates \
-e /home/stack/templates/overcloud_images.yaml \
-e <other templates>
.....
-e /home/stack/templates/dellemc-backend-env.yaml  \
```

### Multiple Backend Deployment
Multiple backends can be deployed at once (same kind of different kinds), The following sample environment file defines two PowerMax back ends, tripleo_emc_powermax1 using iSCSI driver and tripleo_emc_powermax2 using the FC driver:

```yaml
parameter_defaults:
  ControllerExtraConfig:
    cinder::config::cinder_config:
      tripleo_dellemc_powermax1/volume_driver:
        value: cinder.volume.drivers.dell_emc.powermax.iscsi.PowerMaxISCSIDriver
      tripleo_dellemc_powermax1/volume_backend_name:
        value: tripleo_dellemc_powermax1
      tripleo_dellemc_powermax1/san_ip:
        value: '10.10.10.10'
      tripleo_dellemc_powermax1/san_login:
        value: 'my_username'
      tripleo_dellemc_powermax1/san_password:
        value: 'my_password'
      tripleo_dellemc_powermax1/vmax_port_groups:
        value: '[OS-ISCSI-PG]'
      tripleo_dellemc_powermax1/vmax_array:
        value: '000123456789'
      tripleo_dellemc_powermax1/vmax_srp:
        value: 'SRP_1'
      tripleo_dellemc_powermax2/volume_driver:
        value: cinder.volume.drivers.dell_emc.powermax.fc.PowerMaxFCDriver
      tripleo_dellemc_powermax2/volume_backend_name:
        value: tripleo_dellemc_powermax2
      tripleo_dellemc_powermax2/san_ip:
        value: '10.10.10.10'
      tripleo_dellemc_powermax2/san_login:
        value: 'my_username'
      tripleo_dellemc_powermax22/san_password:
        value: 'my_password'
      tripleo_dellemc_powermax2/vmax_port_groups:
        value: '[OS-FC-PG]'
      tripleo_dellemc_powermax2/vmax_array:
        value: '000123456789'
      tripleo_dellemc_powermax2/vmax_srp:
        value: 'SRP_1'
    cinder_user_enabled_backends: ['tripleo_dellemc_powermax1', 'tripleo_dellemc_powermax2']
```
Do not use **cinder_user_enabled_backends** to list back ends that you can enable natively with director. 
For more information see the Red Hat [Custom Block Storage Back End Deployment Guide](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/16.0/html/custom_block_storage_back_end_deployment_guide/envfile)

### Verify the configured changes

When the director completes the overcloud deployment, check that the volume services are up using the openstack cli command. You can also verify that the cinder.conf in the cinder container and it should reflect changes made above.
``` bash
$ openstack volume service list
```
### Testing the configured Backend
After you deploy the back ends to the overcloud, create a volume-type per backend and test if you can successfully create and attach volumes of that type.

## References
* [Red Hat OpenStack Platform Overcloud Custom Block Storage Backend Guide](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/17.0/html/custom_block_storage_back_end_deployment_guide/index)

  





