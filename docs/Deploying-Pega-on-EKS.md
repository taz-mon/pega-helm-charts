Deploying Pega Platform on an Amazon EKS cluster
===============================

Deploy Pega Platform™ on an Amazon Elastic Kubernetes Service (Amazon EKS) cluster using an Amazon Relational Database Service (Amazon RDS). These procedures are written for any level of user, from a system administrator to a development engineer who is interested in learning how to install and deploy Pega Platform onto a EKS cluster.

Pega helps enterprises and agencies quickly build business apps that deliver the outcomes and end-to-end customer experiences that you need. Use the procedures in this guide, to install and deploy Pega software onto a EKS cluster without much experience in either EKS configurations or Pega Platform deployments.

Create a deployment of Pega Platform on which you can implement a scalable Pega application in a EKS cluster. You can use this deployment for a Pega Platform development environment. By completing these procedures, you deploy Pega Platform on a EKS cluster with a Amazon RDS database instance and two clustered virtual machines (VMs).

Deployment process overview
------------------------

Use Kubernetes tools and the customized orchestration tools and Docker images to orchestrate a deployment in a EKS cluster that you create for the deployment:

1. Prepare your local system:

    - To prepare a local Linux system, install required applications and configuration files - [Preparing your local Linux system – 45 minutes](prepping-local-system-runbook-linux.md).

    - To prepare a local Windows system, install required applications and configuration files - [Preparing your local Windows 10 system – 45 minutes](prepping-local-system-runbook-windows.md).

2. Create an Amazn EKS cluster and create an Amazon RDS instance in your AWS account - [Prepare your Amazon EKS resources – 45 minutes](#prepare-your-resources--45-minutes).

3. Customize a configuration file with your Amazon EKS cluster details and use the command-line tools, AWS CLI, eksctl, kubectl and Helm, to install and then deploy Pega Platform onto your Amazon EKS cluster - [Deploying Pega Platform using Helm charts – 90 minutes](#installing-and-deploying-pega-platform-using-helm-charts--90-minutes).

4. Configure your network connections in the DNS management zone of your choice so you can log in to Pega Platform - [Logging into Pega Platform – 10 minutes](#logging-into-pega-platform--10-minutes).

To understand how Pega maps Kubernetes objects with Pega applications and services, see [Understanding the Pega deployment architecture](https://community.pega.com/knowledgebase/articles/client-managed-cloud/cloud/understanding-pega-deployment-architecture).

Assumptions and prerequisites
-----------------------------

This guide assumes:

- You have a basic familiarity with running commands from a Windows 10 PowerShell with Administrator privileges or a Linux command prompt with root privileges.

- You use open source packaging tools on Windows or Linux to install applications onto your local system.

The following account, resources, and application versions are required for use in this document:

- An Amazon AWS account with a payment method set up to pay for the Amazon cluster and RDS resources you create and appropriate AWS account permissions and knowledge to:

  - Create an Amazon RDS database.

  - Select an appropriate location in which to deploy your database resource; the document assumes your location is US East.

  You are responsible for any financial costs incurred for your AWS resources.

- Pega Platform 8.3.1 or later.

- Pega Docker images – your deployment requires the use of several Docker images that you download and make available in a private Docker registry. For details, see [Downloading Docker images for your deployment](https://github.com/pegasystems/pega-helm-charts#downloading-docker-images-for-your-deployment).

- Helm 3.0 or later. Helm is only required to use the Helm charts and not to use the Kubernetes YAML examples directly. For more information, see the [Helm documentation portal](https://helm.sh/docs/).

- kubectl – the Kubernetes command-line tool that you use to connect to and manage your Kubernetes resources.

- AWS IAM Authenticator for Kubernetes - the AWS command-line tool that you use to configure the required AWS CLI credentials for deploying your Amazon EKS cluster: access key, secret access key, AWS Region, and output format.

- eksctl - the Amazon EKS command-line tool that you use for creating and managing clusters on Amazon EKS. If you have to update the version of kubernetes on an existing Amazon EKS cluster, enter the following command, replacing `<cluster-name>` with your existing cluster name: `eksctl update cluster --name <cluster-name> --approve`.

Prepare your Amazon EKS resources – 45 minutes
----------------------------------------------

This section covers the details necessary to obtain your AWS credentials and configure the required Amazon RDS database in an AWS account. Pega supports creating a database resource in any environment if the IP address of the database is available to your Amazon EKS cluster.

### Creating your IAM user access keys

In order to create an EKS cluster, you must create your IAM user access keys. Use the following steps, which are sourced from the AWS documentation, **Access Key and Secret Access Key** on the page, [Quickly Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html#cli-quick-configuration).

To create access keys for an IAM user

1. Sign in to the AWS Management Console and open the IAM console at https://console.aws.amazon.com/iam/.

2. In the navigation pane, choose Users.

3. Choose the name of the user whose access keys you want to create, and then choose the Security credentials tab.

4. In the Access keys section, choose Create access key.

5. To view the new access key pair, choose Show. You will not have access to the secret access key again after this dialog box closes. Your credentials will look something like this:

- Access key ID: AKIAIOSFODNN7EXAMPLE
- Secret access key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

6. To download the key pair, choose Download .csv file. Store the keys in a secure location. You will not have access to the secret access key again after this dialog box closes.

Keep the keys confidential in order to protect your AWS account and never email them. Do not share them outside your organization, even if an inquiry appears to come from AWS or Amazon.com. No one who legitimately represents Amazon will ever ask you for your secret key.

7. After you download the .csv file, choose Close. When you create an access key, the key pair is active by default, and you can use the pair right away.

### Configuring your AWS CLI credentials

To save your IAM user access keys and other preferences for your EKS deployment to a configuration file on your local system, use the `aws configure` command. this command prompts you for four pieces of information you must specify in order to deploy an EKS cluster in your AWS account (access key, secret access key, AWS Region, and output format). For complete details about what this information includes, see the overview article, [Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html). To see details that may be useful to customize your stored credentials to meet your organization's business needs, see [Configuration and Credential File Settings](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html).

To setup your local system and save your AWS credentials and profile to the $USER/.aws file, enter:

`$ aws configure`

You are prompted for your AWS access credentials and details. Enter your own values. For guidance on completing each value, see [Quickly Configuring the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html#cli-quick-configuration).

AWS Access Key ID [None]: your-key-ID
AWS Secret Access Key [None]: your-secrete-access-key
Default region name [None]: your-region-preference
Default output format [None]:  specify your preference for a result format.

With your credentials saved locally, you must push your Pega-provided Docker images to your Docker registry. For details on where the AWS CLI stores your credentials locally, see [Where Are Configuration Settings Stored?](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html#cli-configure-files-where)

### Making your Docker images available to your deployment

TBD




With your Docker images available to your AWS account, you are ready to create your Amazon EKS cluster.

### Creating an Amazon EKS cluster

You can create your EKS cluster using the `eksctl` command line utility. This example shows how to define your configuration in a yaml file that you pass to the `eksctl` command. For more details and options available to advanced EKS users, see [Create your Amazon EKS cluster and worker nodes](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html#eksctl-create-cluster).

At a minimum, your cluster must be provisioned with at least two worker nodes that have 32GB of RAM in order to support the typical processing loads in a Pega
Platform deployment; for this option, Pega recommends using a minimum of two m5.xlarge nodes for your deployment. Pega has not tested this method using Windows worker nodes.

To create Amazon EKS cluster with Linux worker nodes:

1. Save the following text to a file named similar to `my-EKS-cluster-spec.yaml` in your EKS-demo folder:

```yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
name: pega-84-demo
region: us-east-1
version: '1.15'

nodeGroups:
- name: linux-ng
instanceType: m5.xlarge
minSize: 3
```

2. To create your Amazon EKS cluster and Windows and Linux worker nodes, from your /EKS-demo folder, enter.

`eksctl create cluster -f ./cluster-spec.yaml`

 It takes 10 to 15 minutes for the cluster provisioning to completion. During deployment the required Kubernetes configuration file is copied into the cluster and into your default ~/.kube directory.
 
 3. After provisioning is complete, to verify that the worker nodes joined the cluster and are in Ready state, enter:

`kubectl get nodes`

With your cluster created and running as expected, you must create a database resource for your Pega Platform installation. If you have to delete your cluster for some reason before you have namespaces deployed in it, use the command, `eksctl delete cluster --name <cluster-name>'.

4. To deploy the Kubernetes dashboard and see your EKS cluster, enter:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta4/aio/deploy/recommended.yaml

5. Create an eks-admin Service Account and Cluster Role Binding to securely connect to the dashboard with admin-level permissions as by default Kubernetes dashboard has limited permissions.

Create a file called `eks-admin-service-account.yaml` using the following example text.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: eks-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: eks-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
-kind: ServiceAccount
 name: eks-admin
 namespace: kube-system
```

This manifest defines a service account and cluster role binding called `eks-admin`.

6. Apply the service account and cluster role binding to your cluster.

`kubectl apply -f eks-admin-service-account.yaml`

7. Retrieve an authentication token for the eks-admin service account. Copy the <authentication_token> value from the output. You use this token to connect to the dashboard. 

tbd

8. Connect to the dashboard using the administrative service account.

`kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep eks-admin | awk '{print $1}')`

9. To start the proxy server for the Kubernetes dashboard, enter:

    `$ kubectl proxy`

10. To access the Dashboard UI, open a web browser and navigate to the following URL:

    `http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/`

11. In the **Kubernetes Dashboard** sign in window, choose the appropriate authentication method:

- To use a cluster Kubeconfig access credential file: select **Kubeconfig**, navigate to your \<local filepath\>/.kube directory and select the config file. Click **SIGN IN**.

- To use a cluster a Kubeconfig token: select **Token** and paste your Kubeconfig token into the **Enter token** area. Click **SIGN IN**.

    You can now view your deployment details using the Kubernetes dashboard. Use this dashboard to review the status of your deployment. Without a deployment, only GKE resources display. The dashboard does not display your GKE cluster name or your resource name, which is expected behavior.

    To continue using the Kubernetes dashboard to see the progress of your deployment, keep this shell open. At this point, you must create a database instance into which you must install Pega Platform.

### Creating a database resource

AMAZON EKS deployments require you to install Pega Platform software in an Amazon RDS instance that contains a PostgreSQL database. After you create an SQL instance that is available to your EKS cluster, you must create a PostgreSQL database in which to install Pega Platform. When you are finished, you will need the database name and the SQL instance IP address which you create in this procedure in order to add this information to your pega.yaml Helm chart.

#### Creating an Amazon RDS instance

Create a database that is available to your EKS cluster. In this example, we create an Amazon RDS instance in AWS; however, you can create or use an database resource that is available to your EKS cluster. Pega Platfrom deployments require this region is the same as the region where your EKS worker nodes are located.

1. Use a web browser to log in to <https://aws.amazon.com/console/>, which logs you into your default region.

2. In the AWS Management Console, use the search tool to navigate to **Amazon RDS** main page.

3. In **Amazon RDS** main page, in the Create Database section, click **Create database**.

    This process will create an RDS database in your default region. If this region is different from the region where your EKS worker nodes are located, you must reset your region to the region when you created your EKS cluster.

4. To create your database, in the **Create database** page, select the following options:

   a. For **Choose a database creation method**, select **Standard create**.
   
   b. For **Engine Options**, select **PostgreSQL**.

   c. For **Version**, select the latest available PostgreSQL version 11 series.

   d. For **Templates**, select **Production**.

   e. In the **Settings** section, add details:

   - In the **DB instance identifier**, enter a unique \<*databasename*\>.
   - In **Credentials Settings**, leave **Master username** set to the default.
   - Create a master password for your database that meets your organization standards.

   f. In the **DB instance size** section, add details:

   - Select **Standard classes**.
   - Select **b.m5.2xlarge** or greater. The **b.m5.2xlarge** selection  provides the minimum hardware requirement for Pega Platform installations of a minimum of **4** vCores and **32 GB** storage.

   g. In the **Storage** section, for details, accept the default values:

   - **Storage type** is **Provisioned IOPS**.
   - **Allocated storage** is **100GiB**.
   - **Provisioned IOPS** is **1000**.
   - **Storageautoscaling** can be selected.


    e. In **Configuration options \> Connectivity**, select **Public IP**, click **+ Add Network**, enter a **Name** and **Network** of one or more IP address to whitelist for this PostgreSQL database, and click **Done**.

    For clusters that are provisioned by Pivotal: you can launch the Kubernetes dashboard to view the external IP address of the nodes in your cluster; to add that IP network to the database whitelist, enter the first three sets of number,and use 0/24 for the final set in this IP range. For example: 123.123.123.0/24.

6. In **Configuration options \> Machine type and storage**:

    a. For **Machine type**, select 4 vCPU **Cores** and 15 GB **Memory**.

    b. For **Network throughput**, select **SSD (Recommended)**.

    c. For **Storage capacity**, enter **20 GB** and select **Enable automatic storage increases**.

7. Configure the remaining setting using the default values:

    a. For **Auto backups and high availability**, select backups can be automated 1AM – 5AM in a single zone.

    b. For **Flags**, no flags are required.

    c. **For Maintenance**, any preference is supported.

    d. For **Labels**, no labels are required.
    
    Labels can help clarify billing details for your PKS resources.

8. Click **Create**.

    A deployment progress page displays the status of your deployment until it is complete, which takes up to 5 minutes. When complete, the GCP UI displays all of the SQL resources in your account, which includes your newly created SQL instance:

![cid:image007.png\@01D5A3B1.62812F70](media/9aa072ea703232c2f6651fe95854e8dc.62812f70)

#### Creating a database in your SQL instance

Create a PostgreSQL database in your new SQL instance for the Pega Platform installation. Use the database editing tool of your choice to log into your SQL instance and create this new PostgreSQL database. The following example was completed using pgAdmin4.

1. Use a database editor tool, such as pgadmin4, to log into your SQL instance.

    You can find your access information and login credentials, by selecting the SQL instance in the GCP console.

2. In the database editor tool, navigate to Databases and create a new database.

   No additional configuration is required.

With your SQL service IP address and your new database name, you are ready to continue to the next section.

Installing and deploying Pega Platform using Helm charts – 90 minutes
---------------------------------------------------------------------

To deploy Pega Platform by using Helm, customize the pega.yaml Helm chart that holds the specific settings for your deployment needs and then run a series of Helm commands to complete the deployment.

An installation with deployment will take about 90 minutes total, because a Pega Platform installation in your PostgreSQL database takes up to an hour.

### Updating the pega.yaml Helm chart values

To deploy Pega Platform, configure the parameters in the pega.yaml Helm chart to your deployment resource. Pega maintains a repository of Helm charts that are required to deploy Pega Platform by using Helm, including a generic version of this chart. To configure parameters this file, download it from the repository to your local system, edit it with a text editor, and then save it with the same filename. To simplify the instruction, you can download the file to the \gke-demo folder you have already created on your local system. 

Configure the parameters so the pega.yaml Helm chart matches your deployment resources in these areas:

- Specify that this is an PKS deployment.

- Credentials for your DockerHub account in order to access the required Docker images.

- Access your GCP SQL database.

- Install the version of Pega Platform that you built into your Docker installation image.

- Specify host names for your web and stream tiers.

1. To download the pega.yaml Helm chart to the \<local filepath\>/pks-demo, enter:

`$ helm inspect values pega/pega > pega.yaml`

2. Use a text editor to open the pega.yaml file and update the following parameters in the chart based on your PKS requirements:

| Chart parameter name    | Purpose                                   | Your setting |
|-------------------------|-------------------------------------------|--------------|
| provider:               | Specify a PKS deployment.                 | provider:"pks"|
| actions.execute:        | Specify a “deploy” deployment type.       | execute: "deploy"   |
| Jdbc.url:               | Specify the database IP address and database name for your Pega Platform installation.        | <ul><li>url: "jdbc:postgresql://**localhost**:5432/**dbName**"</li><li>where **localhost** is the public IP address you configured for your database connectivity and **dbName** is the name you entered for your PostgreSQL database in [Creating a database resource](#creating-a-database-resource).</li></ul>|
| Jdbc.driverClass:       | Specify the driver class for a PostgreSQL database. | driverClass: "org.postgresql.Driver"                |
| Jdbc.dbType:            | Specify PostgreSQL database type.         | dbType: "postgres”   |
| Jdbc.driverUri:         | Specify the database driver Pega Platform uses during the deployment.| <ul><li>driverUri: "latest jar file available” </li><li>For PostgreSQL databases, use the URL of the latest PostgreSQL driver file that is publicly available at <https://jdbc.postgresql.org/download.html>.</li></ul>|
| Jdbc: username: password: | Set the security credentials for your database server to allow installation of Pega Platform into your database.   | <ul><li>username: "\<name of your database user\>" </li><li>password: "\<password for your database user\>"</li><li>-- For GCP PostgreSQL databases, the default user is “postgres”.</li></ul>|
| jdbc.rulesSchema: jdbc.dataSchema:  | Set the names of both your rules and the data schema to the values that Pega Platform uses for these two schemas.      | rulesSchema: "rules" dataSchema: "data" |
| docker.registry.url: username: password: | Map the host name of a registry to an object that contains the “username” and “password” values for that registry. For more information, search for “index.docker.io/v1” in [Engine API v1.24](https://docs.docker.com/engine/api/v1.24/). | <ul><li>url: “<https://index.docker.io/v1/>” </li><li>username: "\<DockerHub account username\>"</li><li> password: "\< DockerHub account password\>"</li></ul> |
| docker.pega.image:       | Refer to the latest Pega Platform deployment image on DockerHub.  | <ul><li>Image: "pegasystems/pega:latest" </li><li>For a list of default images that Pega provides: <https://hub.docker.com/r/pegasystems/pega-ready/tags></li></ul> |
| upgrade:    | Do not set for installations or deployments. | upgrade: for non-upgrade, keep the default value. |
| tier.name: ”web” tier.service.domain:| Set a host name for the pega-web service of the DNS zone. | <ul><li>domain: "\<the host name for your web service tier\>" </li><li>Assign this host name with an external IP address and log into Pega Platform with this host name in the URL. Your web tier host name must comply with your networking standards and be available as an external IP address.</li></ul>|
| tier.name: ”stream” tier.service.domain: | Set the host name for the pega-stream service of the DNS zone.   | <ul><li>domain: "\<the host name for your stream service tier\>" </li><li>Your stream tier host name should comply with your networking standards. </li></ul>|
| installer.image:        | Specify the Docker image you built to install Pega Platform. | <ul><li>Image: "\<your installation Docker image :your tag\>" </li><li>You created this image in  [Preparing your local Linux system](docs/prepping-local-system-runbook-linux.md)</li></ul>|
| installer. adminPassword:                | Specify a password for your initial log in to Pega Platform.    | adminPassword: "\<initial password\>"  |

3. Save the file.

### Deploy Pega Platform using the command line

A Helm installation and a Pega Platform installation are separate processes. The Helm install command uses Helm to install your deployment as directed in the Helm charts, one in the **charts\\addons** folder and one in the **charts\\pega** folder.

In this document, you specify that the Helm chart always “deploys” by using the setting, actions.execute: “deploy”. In the following tasks, you overwrite this function on your *initial* Helm install by specifying `--set global.actions.execute:install-deploy`, which invokes an installation of Pega Platform using your installation Docker image and then
automatically followed by a deploy. In subsequent Helm deployments, you should not use the override argument, `--set global.actions.execute=`, since Pega Platform is already installed in your database.

1. Do one of the following:

- Open Windows PowerShell running as Administrator on your local system and change the location to the top folder of your pks-demo folder that you created in [Preparing your local Windows 10 system](https://github.com/pegasystems/pega-helm-charts/blob/master/docs/prepping-local-system-runbook-windows.md).

`$ cd <local filepath>\pks-demo`

- Open a Linux bash shell and change the location to the top folder of your pks-demo directory that you created in [Preparing your local Linux system](https://github.com/pegasystems/pega-helm-charts/blob/master/docs/prepping-local-system-runbook-linux.md).

`$ cd /home/<local filepath>/pks-demo`

2. Use the PKS CLI to log into your account using the Pivotal-provided API and login credentials and skip SSL validation.

`$ pks login -a <API> -u <USERNAME> -p <PASSWORD> -k`

If you need to validate with SSL, replace the -k with --ca-cert \<PATH TO CERT\>.

3. View the status of all of your PKS clusters and verify the name of the cluster for the Pega Platform deployment.

`$ pks clusters`

Your cluster name is displayed in the **Name** field.

4. To use the PKS CLI to download the cluster Kubeconfig access credential file, which is specific to your cluster, into your \<local filepath\>/.kube directory.

```yaml
$ pks get-credentials <cluster-name>`
Fetching credentials for cluster pega-platform.
Context set for cluster pega-platform.
```

If you need to use a Bearer Token Access Credentials instead of this credential file, see the Pivotal document, [Accessing Dashboard](https://docs.pivotal.io/pks/1-3/access-dashboard.html).

5. To use the kubectl command to view the VM nodes, including cluster names and status.

`$ kubectl get nodes`

6. Establish a required cluster role binding setting so that you can launch the Kubernetes dashboard.

`$ kubectl create clusterrolebinding dashboard-admin -n kube-system
--clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard`

7. Start the proxy server for the Kubernetes dashboard.

`$ kubectl proxy`

8. To access the Dashboard UI, open a web browser and navigate to the following URL:

<http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/>

9. In the **Kubernetes Dashboard** sign in window, choose the appropriate sign in method:

- To use a cluster Kubeconfig access credential file: select **Kubeconfig**, navigate to your \<local filepath\>/.kube directory and select the config file. Click **SIGN IN**.

- To use a cluster a Kubeconfig token: select **Token** and paste your Kubeconfig token into the **Enter token** area. Click **SIGN IN**.

    You can now view your deployment details using the Kubernetes dashboard. Use this dashboard to review the status of your deployment. Without a deployment, only PKS resources display. The dashboard does not display your PKS cluster name or your resource name, which is expected behavior.

    To continue using the Kubernetes dashboard to see the progress of your deployment, keep this PowerShell or Linux shell open.

10. Do one of the following:

- Open a new Windows PowerShell running as Administrator on your local system and change the location to the top folder of your pks-demo folder.

`$ cd <local filepath>\pks-demo`

- Open a new Linux bash shell and change the location to the top folder of your pks-demo directory.

`$ cd /home/<local filepath>/pks-demo`

11. Create namespaces in preparation for the pega.yaml and addons.yaml deployments.

```yaml
$ kubectl create namespace mypega-pks-demo
namespace/mypega-pks-demo created
$ kubectl create namespace pegaaddons
namespace/pegaaddons created
```

12. Install the addons chart, which you updated in [Preparing your local system](#Prepare-your-local-system-–-45-minutes).

```yaml
$ helm install addons pega/addons --namespace pegaaddons --values addons.yaml
```

The `pegaddons` namespace contains the deployment’s load balancer and the metric server configurations that you configured in the addons.yaml Helm chart. A successful pegaaddons deployment returns details of deployment progress. For further verification of your deployment progress, you can refresh the Kubernetes dashboard and look in the `pegaaddons` **Namespace** view.

13. Deploy Pega Platform for the first time by specifying to install Pega Platform into the database specified in the Helm chart when you install the pega.yaml Helm chart.

```yaml
helm install mypega-pks-demo pega/pega --namespace mypega-pks-demo --values pega.yaml --set global.actions.execute=install-deploy
```

For subsequent Helm installs, use the command `helm install mypega-pks-demo pega/pega --namespace mypega-pks-demo` to deploy Pega Platform and avoid another Pega Platform installation.

A successful Pega deployment immediately returns details that show progress for your `mypega-pks-demo` deployment.

14. Refresh the Kubernetes dashboard that you opened in step 8. If you closed the dashboard, start the proxy server for the Kubernetes dashboard as directed in Step 7, and relaunch the web browser as directed in Step 8.

15. In the dashboard, in **Namespace** select the `mypega-pks-demo` view and then click on the **Pods** view. Initially, you can some pods have a red status, which means they are initializing:

![](media/dashboard-mypega-pks-demo-install-initial.png)

    Note: A deployment takes about 15 minutes for all resource configurations to initialize; however a full Pega Platform installation into the database can take up to an hour.

    To follow the progress of an installation, use the dashboard. For subsequent deployments, you do not need to do this. Initially, while the resources make requests to complete the configuration, you will see red warnings while the configuration is finishing, which is expected behavior.

16. To view the status of an installation, on the Kubernetes dashboard, select **Jobs**, locate the **pega-db-install** job, and click the logs icon on the right side of that row.

    After you open the logs view, you can click the icon for automatic refresh to see current updates to the install log.

17. To see the final deployment in the Kubernetes dashboard after about 15 minutes, refresh the `mypega-pks-demo` namespace pods.

![](media/f7779bd94bdf3160ca1856cdafb32f2b.png)

A successful deployment does not show errors across the various workloads. The `mypega-pks-demo` Namespace **Overview** view shows charts of the percentage of complete tiers and resources configurations. A successful deployment has 100% complete **Workloads**.

![](media/0fb2d07a5a8113a9725b704e686fbfe6.png)

Logging in to Pega Platform – 10 minutes
---------------------------------------

After you complete your deployment, as a best practice, associate the host name of the pega-web tier ingress with the IP address that the deployment load balancer assigned to the tier during deployment. The host name of the pega-web tier ingress used in this demo, **pks.web.dev.pega.io**, is set in the pega.yaml file in the following lines:

```yaml
tier:
  - name: "web"

    service:
      # Enter the domain name to access web nodes via a load balancer.
      #  e.g. web.mypega.example.com
      domain: "**pks.web.dev.pega.io**"
```

To log in to Pega Platform with this host name, assign the host name with the same IP address that the deployment load balancer assigned to the web tier. This final step ensures that you can log in to Pega Platform with your host name, on which you can independently manage security protocols that match your networking infrastructure standards.

To manually associate the host name of the pega-web tier ingress with the tier endpoint, use the DNS lookup management system of your choice. As an example, if your organization has a GCP **Cloud DNS** that is configured to manage your DNS lookups, create a record set that specifies the pega-web tier the host name and add the IP address of the pega-web tier.

For GCP **Cloud DNS** documentation details, see [Quickstart](https://cloud.google.com/dns/docs/quickstart). If not using the GCP **Cloud DNS**, for configuration details, see the documentation for your DNS lookup management system.

With the ingress host name name associated with this IP address in your DNS service, you can log in to Pega Platform with a web browser using the URL: http://\<pega-web tier ingress host name>/prweb.

![](media/25b18c61607e4e979a13f3cfc1b64f5c.png)