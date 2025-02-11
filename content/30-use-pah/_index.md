+++
title = "Using Private Automation Hub"
weight = 30
+++

## Using Private Automation Hub

### Prerequisites

* Automation Controller is running and accessible

* Private Automation Hub is running and accessible

### Tasks

* Configure Automation Controller to access your Private Automation Hub

* Add collections from Red Hat Automation Hub, Galaxy and custom ones created by you.

* Add execution environment images

Let's start, as the docs for this are distributed over some places we'll give some more instructions.

### Add content to your private Automation Hub

Most of this is well documented [here](https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.3/html-single/managing_red_hat_ansible_content_collections_and_ansible_galaxy_collections_in_automation_hub/index)

#### Sync collections from Red Hat Automation Hub

* Go to `console.redhat.com` and open **Ansible Automation Platform** -> **Automation Hub** -> **Collections**. Here you can enable/disable the sync of certain collections.

* What you need to do is to get the authentication token and configure it in your PAH:

  * In Red Hat Automation Hub go to **Connect to Hub** we will need the **Offline Token** and the **Server URL**

  * In your private automation hub go to **Collections** -> **Repository management** -> **Remote**

  * Edit the `rh-certified` remote:

    * **URL** paste the **Server URL** from the Red Hat Automation Hub

    * **Token** the token you copied from RH AH

    * Click **Save** and then hit **Sync**. This will sync all enabled and updated collections from Red Hat Automation Hub to your Private Automation Hub.

#### Sync selected community collections from Ansible Galaxy

Galaxy is configured as the remote `community` out of the box. Follow the instructions to configure the synchronization.

* Create a regular requirements.yml file pointing to the collection you want, we'll use the one from the docs:

```yaml
collections:
  # Install a collection from Ansible Galaxy.
  - name: containers.podman
```

* Go to Repo Management, click the **Remote** tab again

* Edit the `community` remote

* In **YAML requirements** upload the  `requirements.yml` file from your local machine.

* Click **Save**

* In the **Remote** overview tab click **Sync** for the `community` remote

Verify the sync of the collections in **Collections** -> **Collections**, switch the repository filter with the dropdown at the top. There should be a lot of content in the `Red Hat Certified` repo and one collection in the `Community` repo. The 'published' filter will not find anything, since we haven't uploaded any collections we created ourselves.

### Test Private Automation Hub Integration

Now check that your Automation Controller can actually use the content from your PAH:

* Create a new **Project** pointing here: `https://github.com/ansible-learnfest/ee-flow.git`

  * Have a look at the content, esp the `collections/requirements.yml` file

* Create an inventory of the target machine or update **Workshop Inventory** by removing `ansible-1` from the list of **Hosts**

* Create a new **Template**:

  * **Name**: up to you

  * **Inventory**: The one you set up with the Playbook before, it should contain one of your AWS instances, or if you updated the `Workshop Inventory`, use this one.

  * **Project**: The one you just created

  * **Execution Environment**: `Ansible Engine 2.9 execution environment`

  * **Playbook**: `deploy-container.yml`

  * Check **Privilege Escalation**

  * Set the right Credentials : `Workshop Credential`

* Launch the **Template**, if all was configured correctly it should deploy an httpd container that is hosting a small website.

So recap what happened:

* You created a **Template** that runs a Playbook that has a requirement on a certain Collection which is not part of the Execution Environments included in Controller.

* Your **Organization** (`default`) is configured in a way it can only download Collections from your Private Automation Hub

* The Collection did exist on your PAH

{{% notice note %}}
As this collection is not part of the Execution Environment the Playbook uses, how did it work? In this case is it was dynamically "added" to the Execution Environment at runtime. This behavior did already exist in Ansible Tower 3.8, and it still does work in automation controller. This means, you only have to build your own execution environment if your collection has additional Python or package dependencies.
{{% /notice %}}

### Goals

Be able to manually configure private automation hub to synchronize content from Red Hat's automation hub and Ansible Galaxy.

You should also better understand that, although it might be beneficial to create custom execution environments, it's not always necessary and automation controller can still load and install collections during runtime.

### Tips

* Check the list of [certified collections](https://access.redhat.com/articles/3642632) in the Red Hat Customer Portal
