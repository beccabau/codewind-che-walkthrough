# Codewind on Che Quicklab 

Moving workloads to the cloud introduces a steep learning curve for developers accustomed to traditional web application development. Cloud-native applications are usually containerized (either Docker or OCI containers) and are run on orchestrated platforms like Kubernetes. These are complex technologies that bring big changes to the development workflow. Can this be simplified? Can developers create cloud-native applicatioons and deploy them to Kubernetes without climbing the mountainous learning curve? 

In this quicklab we will take a look at a use case involving Eclipse Codewind, a technology that simplify development of containerized cloud-native applications. Codewind is an IDE plugin that allows you to work with containerized applications in a familiar way. With Codewind, developers can start building cloud-native applications like experts without having to be an expert in all the cloud-native technologies. 

## Prerequistes

In order to use Codewind as a hosted application on your cloud, make sure your hosted environment has the proper configuration.

### Configure Hosted Environment

This quicklab requires the following configuration: 

1. Set up PersistentVolumes (PVs) that support both ReadWriteOnce (RWO) and ReadWriteMany (RWX) access modes and each have a minimum of 5 Gi storage.
- One volume is required for Che, and two volumes are required for each Codewind workspace.
- For Network File System (NFS) shares, set 777 permissions for the exported folders and ownership of nobody:nogroup.
- Because Codewind uses ReadWriteMany (RWX) volumes to provide persistent storage, use NFS for storage on OpenShift 4.
2. Ensure that the cluster can pull images from the docker.io/eclipse and quay.io/eclipse registries.
- Both Eclipse Che and Eclipse Codewind host Docker images at these locations.
- Many clusters have image policies that control which registries you can use to pull images. Check your cluster documentation and ensure that the cluster image pull policies permit both of these registries.
3. Set up the ClusterRole for Codewind: kubectl apply -f https://raw.githubusercontent.com/eclipse/codewind-che-plugin/0.11.0/setup/install_che/codewind-clusterrole.yaml

### Installing Che with chectl

The fastest way to install Eclipse Che for Codewind is to use the chectl CLI. To install the chectl CLI tool, see [Installing the chectl management tool](https://www.eclipse.org/che/docs/che-7/installing-the-chectl-management-tool/).

After you install chectl, download the [codewind-checluster.yaml](https://raw.githubusercontent.com/eclipse/codewind-che-plugin/0.11.0/setup/install_che/che-operator/codewind-checluster.yaml) file.

- You can modify this file, but leave the spec.server.cheWorkspaceClusterRole field set to eclipse-codewind and the spec.storage.preCreateSubPaths field set to true.

#### Installing on OpenShift

<details>
  <summary>Click to expand</summary>

Eclipse Che on OpenShift makes use of the router’s existing certificates. Run the following command to install Che on OpenShift with chectl:

   $ chectl server:start --platform=openshift --installer=operator --che-operator-cr-yaml=codewind-checluster.yaml --che-operator-image=quay.io/eclipse/che-operator:7.9.2

</details>

#### Installing on Kubernetes

<details>
  <summary>Click to expand</summary>

1. Create the che namespace if it doesn’t already exist: kubectl create namespace che.
2. Determine your Ingress domain.
- Set the spec.server.ingressDomain field in the Che .yaml file to the Ingress domain.
- If you’re unsure of your Ingress domain, ask your cluster administrator.
3. Generate TLS certificates and keys. For more information, see [Generating self-signed TLS certificates](https://www.eclipse.org/che/docs/che-7/setup-che-in-tls-mode-with-self-signed-certificate/#generating-self-signed-certificates_setup-che-in-tls-mode-with-self-signed-certificate).
4. Generate a Kubernetes secret containing the TLS secret and key you generated in the previous set:
```
$ kubectl create secret tls che-tls --key=domain.key --cert=domain.crt -n che
```
5. Generate a Kubernetes secret containing the certificate you generated in step 2:
```
$ cp rootCA.crt ca.crt
$ kubectl create secret generic self-signed-certificate --from-file=ca.crt -n che
```
6. In the ```codewind-checluster.yaml``` file, set ```tlsSecretName: 'che-tls'```
7. Run the following command to install Che:
```
$ chectl server:start --platform=k8s --installer=operator --domain=<ingress-domain> --che-operator-cr-yaml=codewind-checluster.yaml --che-operator-image=quay.io/eclipse/che-operator:7.9.2
```

</details>

#### Updating an existing Che installation

<details>
  <summary>Click to expand</summary>
If you already have a Che installation with TLS, you can update it for Codewind.

Run the following command, where $NAMESPACE is the namespace that your Che workspaces run in. By default, this namespace is che.

```
$ kubectl apply -f https://raw.githubusercontent.com/eclipse/codewind-che-plugin/0.11.0/setup/install_che/codewind-clusterrole.yaml -n $NAMESPACE
```
</details>

## Improving Developer Productivity with Eclipse Codewind

[Eclipse Codewind](https://www.eclipse.org/codewind/) is a plugin for IDEs, currently available in Eclipse Che, Eclipse, VS Code, and IntelliJ, that helps improve developer productivity when developing containerized applications. Let's explore how Codewind can help you be a more productive developer.

