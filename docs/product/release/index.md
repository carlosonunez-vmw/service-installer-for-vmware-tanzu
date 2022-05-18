# Service Installer for VMware Tanzu 1.1

Service Installer for VMware Tanzu automates the deployment of the reference designs for Tanzu for Kubernetes Operations on the following platforms:

- Tanzu Kubernetes Grid on VMware Cloud
- Tanzu Kubernetes Grid on vSphere with NSX-T
- Tanzu Kubernetes Grid on vSphere running Virtual Distributed Switch (VDS)
- Tanzu Kubernetes Grid Service on vSphere running Virtual Distributed Switch (VDS)

For information about the reference designs for Tanzu for Kubernetes Operations, see [VMware Tanzu Reference Architecture Documentation](https://docs.vmware.com/en/VMware-Tanzu-Reference-Architecture/index.html).

Service Installer simplifies the deployment of a Kubernetes environment. It uses best practices for deploying and configuring the required Tanzu for Kubernetes Operations components, such as:

- Tanzu Kubernetes Grid
- NSX Advanced Load Balancer
- Shared services, such as Contour, Harbor, FluentBit, Prometheus, Grafana
- Integration with Tanzu Mission Controller

## Release Notes
See the [Release Notes](WhatsNew.md) for a summary of what's new in this release.

## Download Service Installer for VMware Tanzu
For the download location, see [Release Notes](WhatsNew.md).

## Deploy Service Installer for VMware Tanzu
1. Download the Service Installer OVA from **VMware Marketplace**.
2. Deploy the Service Installer OVA on vCenter server via the **Deploy OVF Template** option and provide required computer resources and storage details.
3. Select **Appliance network as Management network** and specify NTP server as well as root password for the VM.

   After the system configuration is completed, the OVA deployment begins.

4. After the deployment is completed, power on the Service Installer for the VMware Tanzu bootstrap VM.

   You can access the Service Installer UI at `http://<Service-Installer-VM-IP>:8888/`.

   To access the Service Installer CLI, log in over SSH. Enter `ssh root@<Service-Installer-VM-IP>`.