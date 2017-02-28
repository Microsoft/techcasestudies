---
layout: post
title: "Implementing A/B testing for Catraca Livre using open-source technologies on Azure"
author: "Caio Chaves Garcez"
author-link: "https://twitter.com/Caio_garcez"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date: 2017-02-20
categories: [DevOps]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: Microsoft and Rivendel joined efforts to provide Brazilian news website Catraca Livre with the means to perform A/B tests for ad placement optimization using OSS technologies on Azure.
language: [English]
verticals: [Communications/Media]
---

![Figure 0 – Catraca Livre Logo]({{ site.baseurl }}/images/2016-12-28-catraca-livre/Figure_0_Catraca_Livre_Logo.png)

This article describes the joint effort conducted by Microsoft and Rivendel to implement an [A/B testing](https://en.wikipedia.org/wiki/A%2FB_testing) process for Catraca Livre, a Brazilian news website. By implementing A/B tests, Catraca Livre will be able to optimize banner and ad placements, maximizing click-through rates (CTR) by 30 percent or more as observed in recent experiments. 

Rivendel is a Brazilian consulting company and a Microsoft Azure Certified partner that focuses on helping its customers migrate applications from on-premises to cloud computing platforms, with a special emphasis on DevOps processes and tools. Catraca Livre currently outsources part of its site operations to Rivendel.

Here is the core team that worked on this project:

- Rivendel
    - Vinícius Cavalcanti, DevOps Team Leader
    - Diogo Santos Lima ([@\_deogo](https://twitter.com/_DEOGO)), Systems Administrator
    - Alexandre Cardoso ([@accbel](https://twitter.com/accbel)), Solution Architect
- Catraca Livre
    - Alexandre de Maio, Technology Manager
    - Amanda Gedra, Developer
    - Rodrigo Godoy ([@rodrigopulse](https://twitter.com/rodrigopulse)), Web Designer
- Microsoft
    - Caio Garcez ([@caio\_garcez](https://twitter.com/Caio_garcez)), Senior Technical Evangelist
    - Vinícius Souza ([@visouza\_br](https://twitter.com/visouza_br)), Senior Technical Evangelist

## Customer profile ##

[Catraca Livre](https://catracalivre.com.br/brasil/) is a Brazilian news web portal focused on free and/or accessible events and activities. The topics they cover include sports, culture, health, education, mobility and entrepreneurship.

They promote events daily for Brazil capitals, including São Paulo, Rio de Janeiro, Belo Horizonte, Brasília, Curitiba, Porto Alegre, Recife and Salvador. They strive to reveal characters, trends and projects to inspire innovative and inclusive communitarian solutions around the world. They aspire to help cities be more civilized, welcoming, and creative.

Catraca Livre was chosen by Deutsche Welle as the best Portuguese language citizenship blog. Their Facebook page has over 8,000,000 followers.

The value stream mapping (VSM) exercise and ensuing hackfest, described in the rest of this case study, were focused on Catraca Livre’s web portal which is their main business asset. They have mobile apps for Android and iOS, but their website is the main venue for publishing news and delivering content.

### Site architecture ###

After a partnership nurtured from Microsoft’s Developer Experience (DX) Brazil team, Catraca Livre migrated their website, previously hosted on AWS, to Azure with the assistance of Rivendel. Figure 1 shows an overview of their current architecture.

*Figure 1 – Catraca Livre site architecture*

![Figure 1 – Catraca Livre site architecture]({{ site.baseurl }}/images/2016-12-28-catraca-livre/Figure_1_Site_Architecture.png)

All VMs run Linux CentOS 6.8. The main components of the site as seen in figure 1 are as follows:

- 2 VMs with Varnish for caching the website content
- 4 VMs with WordPress, PHP-FPM, and nginx for the website itself
- 1 VM for storing session data
- 2 VMs with MySQL for the site back-end
- 2 VMs for storage

### Development and release lifecycle ###

Figure 2 shows a simplified view of the overall code flow at Catraca Livre, beginning with the definition of tasks (new features, backlog items) to be addressed in the next sprint, and ending with the release into production.

*Figure 2 – Catraca Livre development process (simplified)*

![Figure 2 – Catraca Livre development process (simplified)]({{ site.baseurl }}/images/2016-12-28-catraca-livre/Figure_2_Development_Process.png)

The simplified process is as follows:

1. Tasks are grouped into weekly sprints (each sprint will correspond to a new release of their website)
2. Development is done by the internal team and sometimes by a third-party development company
3. Once code is promoted by developers to the next stage, it is reviewed
4. If no issues are found during reviewing, then development artifacts are merged
5. After merging, tests are performed
6. If no issues are found, then code is released into production

The following tools are used in the development, release, and operations process:

- Code and work items are stored in GitHub
- Servers are configured and managed through Ansible
- Jenkins is used to control code promotion and deployment
- Site content is uploaded to the WordPress sites using GlusterFS, which allows all servers to receive the content instantaneously
- All VMs are monitored through Zabbix

## Problem statement ##

Catraca Livre has a small design and development team. They don’t have dedicated resources for Operations and Release roles (these are outsourced to Rivendel). While they don’t suffer the common communication issues that often affect large development and operations teams, their minimal resources makes it even more critical that they adopt optimized processes and tools.

When Catraca Livre was initially approached by Microsoft and Rivendel, they had an idea of their most pressing and critical issues, but to create a common vision of their environment and process we conducted a VSM exercise. VSM is an excellent tool to help technical teams assess their DevOps maturity level and identify bottlenecks, issues, and opportunities for improvement.

*Figure 3 – Value stream mapping*

![Figure 3 – Value stream mapping]({{ site.baseurl }}/images/2016-12-28-catraca-livre/Figure_3_VSM.png)

The VSM exercise provided many insights about Catraca Livre’s processes and tools. Nine different issues were detected, three of which were considered the most critical:

- **Manual testing** – all tests are manual and often performed by the same person who developed the code. Besides consuming a significant amount of time, this inherently contributes to a less than optimal test coverage.
- **Manual merge process** – Catraca’s team works on weekly sprints and there is no continuous integration. This means that once a week they need to manually merge development branches, which is not only time-consuming, but also error-prone.
- **Inability to conduct A/B tests** – Advertising is the main source of revenue for Catraca, so they have a business need to ensure they are optimizing click-through rates (CTR). This means that they often need to figure out the best way to position banners and organize content on their web pages. However, there is no process in place to allow for A/B testing in an easy and controlled manner.

After performing the VSM exercise, Catraca Livre, Microsoft, and Rivendel convened to define how best to address these issues and determine which changes would bring the most short-term benefit. After the discussion, the consensus was the following: 

- Eliminating manual testing requires the creation of unit tests and the adoption of a test framework to run and collect evidence from these tests. This entails considerable effort and should be considered by Catraca’s technical team as an investment that will pay off in the long term.
- Minimizing merge issues is a direct consequence of shorter build cycles. Continuous Integration (CI) is the epitome of this approach, but there are alternatives to teams that prefer a slower pace of integration (like daily builds, for instance). This should be a topic of investigation for the technical team as it demands a significant culture change.
- Providing a way to conduct A/B tests would bring direct business value, so it seemed like the most promising issue to address. Different approaches were discussed, and a strong point was made that it is only by having a very mature and advanced DevOps process that companies can easily and confidently experiment in production. Nevertheless, a solution was found that should be able to address Catraca Livre’s business need.

*Figure 4 – Joint team (Catraca Livre, Rivendel, Microsoft) at the end of the VSM Exercise*

![Figure 4 – Joint team (Catraca Livre, Rivendel, Microsoft) at the end of the VSM Exercise]({{ site.baseurl }}/images/2016-12-28-catraca-livre/Figure_4_VSM_Equipe.jpg)

## Solution, steps, and delivery ##

The plan assembled by the team consisted of the following tasks:

- **Architecture changes**
    1. Provide a means to perform weighted load balancing of web traffic between two different production environments, namely sites “A” and “B” (example: 90 percent to site “A” and 10 percent to site “B”)
    2. Create a second production environment on Azure to run the “B” version of the website
- **Process changes**
    1. Provide an easy way for Catraca’s dev team to publish different versions of their website to each production environment, using Jenkins and Ansible
    2. Map the required changes to their current development process to accommodate A/B tests

Figure 5 shows the new site architecture designed by the joint team (machines in orange are new additions).

*Figure 5 – New site architecture*

![Figure 5 – New site architecture]({{ site.baseurl }}/images/2016-12-28-catraca-livre/Figure_5_New_Site_Architecture.png)

In summary:

- Two new VMs running HAProxy were added to handle weighted load balancing
- Two new Varnish VMs were added to cache content for site “B”
- Two new WordPress/PHP VMs were added to host site “B”
- External and Internal Load Balancing is done exactly as in site “A”
- No changes were made to the data layer

The following sections describe the details of each part of the plan.

### Weighted load balancing with HAProxy ###

As shown in figure 1, the current architecture uses Azure Load Balancing in two instances:

- Externally—to load balance web traffic between two Varnish Servers
- Internally—to load balance traffic coming from the Varnish Servers to the Web Servers themselves

To enable Catraca Livre to perform A/B tests effectively, it is necessary to provide a means to control how much traffic is directed to the A and B versions of their website. In other words, we need to implement weighted load balancing.

The devised solution was to add two new Linux CentOS VMs to the environment, running [HAProxy](http://www.haproxy.org/). HAProxy is an open source TCP/HTTP Load Balancer that is used in several high-traffic websites.

As shown in figure 4, the HAProxy VMs sit between the Internet and the External Azure Load Balancing. Its behavior is configured through a config file, part of which is shown here:

*Code example 1 – HAProxy config file, frontend/backend settings section*

```
#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend haproxy-catracalivre
        bind *:80
        maxconn 50000
        default_backend catracalivre-ambientes

backend catracalivre-ambientes
        balance source
        hash-type consistent

        cookie SITEID insert
        server catracalivre-ambiente-prodA 13.67.209.242:80   weight 50 cookie prodatual check
        server catracalivre-ambiente-prodB    10.9.1.100:80   weight 50 cookie testeab   check

```

We had to configure the following settings to get HAProxy to behave as needed:

- The IP addresses for each production environment (A and B).
- The different weights for each production environment (50/50 in this case).
- Ensure HAProxy handles client affinity so that once a reader is in a given site (A or B) he remains there for a certain period (for example, one week if that is the duration of the A/B test). That is accomplished by the `balance source` and `hash-type consistent` lines.

We also created a simple shell script to facilitate changing the weights whenever desired:

*Code example 2 – Shell script that collects desired weights and applies changes*

```shell-script
#!/bin/bash 
read -p "Provide the desired load balancing weight for the current site (A): " SITE_A

if [[ $SITE_A = ?(+|-)+([0-9]) ]] ; then
        read -p "Provide the desired load balancing weight for the alternative site (B): " SITE_B

                if [[ $SITE_B = ?(+|-)+([0-9]) ]] ; then
                        echo
                        echo "Altering load balancing..."
                        sleep 2

                        cd /etc/haproxy/

                        # Copies the template file to .new 
                        cp haproxy.cfg.template haproxy.cfg.new

                        sed -i s/peso-catraca-atual/$SITE_A/g /etc/haproxy/haproxy.cfg.new
                        sed -i s/peso-catraca-ab/$SITE_B/g /etc/haproxy/haproxy.cfg.new

                        # Moves the current configuration to a backup file
                        mv haproxy.cfg haproxy.cfg.backup
                        mv haproxy.cfg.new haproxy.cfg

                        echo "Change performed successfully!"
                        sleep 2

                        # Restarts service
                        echo
                        echo "Restarting the service..."
                        /etc/init.d/haproxy reload
                        echo "Done!"

                else
                        echo "Please provide a numeric value for Site B!"
                        exit
                fi
else
        echo "Please provide a numeric value for Site A!"
        exit
fi
```

This script backs up the previous configuration (to facilitate rolling back). The script can easily be adapted in the future to receive the desired weights as command line parameters, which would allow this step to be automated as part of a larger orchestration.

Finally, the HAProxy servers were added to the Zabbix monitoring platform.

### Second production environment on Azure ###

The second production environment is shown in orange in figure 5. These VMs were created by using two Azure Resource Manager (ARM) templates:

- One for the 2 HAProxy VMs (which have public IP addresses)
- One for the Varnish and WordPress/PHP VMs (which have internal IP addresses)

The ARM templates do all the work needed to provision the VMs in Azure, including the configuration of the following:

- Storage accounts
- Virtual networks
- Subnets
- Load balancing parameters (external and internal)

The Varnish VMs were quite straightforward to configure: The team only had to reproduce the same settings used for the production environment, by simply changing IP addresses.

*Figure 6 – Vinícius Souza and Diogo configuring the Varnish Servers for site “B”*

![Figure 6 – Vinícius Souza and Diogo configuring the Varnish Servers for site “B”]({{ site.baseurl }}/images/2016-12-28-catraca-livre/Figure_6_Varnish.jpg)

The WordPress/PHP VMs were actually created as a [VM Scale Set](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview), which provides autoscaling. This is not likely to be needed in the short term, as the “B” version shall always receive less traffic (as it is the alternative being tested in contrast to the regular site). But it is something that Catraca Livre and Rivendel intend to adopt for the “A” site as well, which was created as individual VMs and therefore does not have automatic autoscaling.

The team used the [Azure Command-Line Interface](https://github.com/Azure/azure-xplat-cli) (Azure CLI) to generalize and capture a running server from site “A”, taking care to remove the VM Agent first:

*Code example 3 – Generalizing and capturing a Linux VM*

```
waagent -deprovision+user
azure config mode arm
azure vm deallocate -g test-ab -n catraca-vanish-base-testab
azure vm generalize -g myResourceGroup -n myVM
azure vm capture -g myResourceGroup -n myVM -p myVHDNamePrefix -t myTemplate.json
```

When the VHD was ready, the team followed the steps described in link (1) referenced at the end of this post to create the VM itself.

### Jenkins and Ansible integration ###

As mentioned before, Ansible is used for server provisioning and configuration, and Jenkins is used as the orchestration tool for code promotion. It was very important to ensure these tools could continue to be used to manage the new A/B testing process.

The main action here was configuring Jenkins to target the second production environment. Figure 7 shows the Jenkins view of both environments.

*Figure 7 – Jenkins view of two production environments*

![Figure 7 – Jenkins view of two production environments]({{ site.baseurl }}/images/2016-12-28-catraca-livre/Figure_7_Jenkins_1.png)

In the Jenkins dashboard shown in figure 7,

- **Deploy** targets the first (regular) production environment
- **Deploy testeAB** targets the second production environment
- **Rollback** targets the first production environment and deploys the (n-1) version

When clicking the Deploy testeAB link the users see the following view:

*Figure 8 – Jenkins view of the second production environment*

![Figure 8 – Jenkins view of the second production environment]({{ site.baseurl }}/images/2016-12-28-catraca-livre/Figure_8_Jenkins_2.png)

This page allows the user to check different builds, such as the last build and the last successful build.

When a build is performed, Jenkins shows the following output:

*Figure 9 – Jenkins output for builds*

![Figure 9 – Jenkins output for builds]({{ site.baseurl }}/images/2016-12-28-catraca-livre/Figure_9_Jenkins_6.png)

The deployment code for both production environments is the same. Jenkins runs a shell script that activates Ansible to do the actual deployment, using a .YML file for configuration. Here is the code:

*Code example 4 – Build shell script*

```shell-script
#!/usr/bin/env bash

tagansible=$1

if [ "$1"  == "deploy"  ]; then
        cd /var/lib/jenkins/catraca
        tag=$(git fetch && git describe --tags `git rev-list --tags --max-count=1`)

        cd /var/lib/jenkins/ansible/
ansible-playbook \
-i hosts \
--extra-vars "tag=$tag" \
master.yml


#       tag="6.7.3"
elif [ "$1" == "rollback" ]; then
        tag=$2
        cd /var/lib/jenkins/ansible/

        ansible-playbook \
        -i hosts \
        --extra-vars "tag=$tag" \
        master.yml

else
        echo "Choose deploy or rollback"
fi

if [ "$tagansible" == "deploy"  ]; then
        versao=$(ansible -i hosts deploy -m shell -a "cd /var/www/html/repository/catraca && git describe --abbrev=0 --tags" | head -n 2 | tail -n 1)
        echo "deploy versio=$versao"
elif [ "$tagansible" == "rollback" ]; then
        versao=$(ansible -i hosts deploy -m shell -a "cd /var/www/html/repository/catraca/ && git describe --abbrev=0 --tags" | head -n 2 | tail -n 1)
        echo "rollback version=$versao"
else
        echo "review deployment script"

```

This script handles both deployment and rollback. Depending on user choice (or in this case, which link was clicked on Jenkins), the script sets the tag variable accordingly and starts Ansible with the master.yml file, shown here:

*Code example 5 – Ansible master.yml file*

```yml
- name: Deploy CatracaLivre
    hosts: deploy
    user: rivendel
    vars:
      dir: /var/www/html/repository/catraca

    tasks:
    - name: git
      shell: "git fetch"
      args:
        chdir: "{{ dir }}"

    - name: deploy
      shell: sudo git reset --hard "{{ tag }}"
      args:
        chdir: "{{ dir }}"
```

As can be seen, this is a pretty straightfoward file—it simply sets some parameters and defines the git and deploy tasks, which do a git fetch and a git reset, respectively.

### Changes to the development lifecycle ###

The joint team extensively discussed different approaches to implementing A/B testing from a development process perspective. Considering that there is no CI and no Automated Testing, generating the “B” version of the site while simultaneously conducting regular development work did not seem viable. Therefore, the proposed process is:

1. At the beginning of each sprint, Catraca Livre decides if they will work on site features and bug fixes or if they will prepare the site for an A/B test
    1. In the first case, nothing changes and the development process is the one shown in figure 2
    2. In the second case, the team follows steps 2–6
2. The team creates a branch in git to work on the “B” version of the site
3. When work on the “B” site is done it is tested and any issues are addressed
4. The second production environment is provisioned using Jenkins and Ansible
5. The “B” site is deployed to this production environment
6. Data is collected and analyzed to assess which version performed better:
    1. If “A” performs better, changes made to “B” are ignored
    2. If “B” performs better, the “B” site is merged into the main git branch

## Conclusion ##

Catraca Livre has an average of 2,000,000 page views per day, and each visitor spends an average of 3½ minutes on the site. The site has a network of more than 20 marketing partners that often create ads and campaigns, and are Catraca’s main source of revenue. **A/B tests will help Catraca ensure they optimize banner and ad placement to maximize CTR**. A recent experiment, performed manually before the implementation of the A/B testing process, **showed an increase of 30 percent in CTR which directly translated into increased revenue**.

The process implemented by Microsoft and Rivendel will allow Catraca Livre to perform A/B tests with confidence and control, even if the current development process does not incorporate CI and Automated Testing. Once these processes are in place, Catraca Livre can become even more agile in the way they handle bug fixes, new feature development, and A/B testing in a simultaneous or near-simultaneous way.

Catraca Livre’s expectation is that they will be able to reap business value in the very short term, while at the same time improving their processes and continuing to evolve on their DevOps journey.

*Figure 10 – Microsoft and Rivendel teams at the end of the 3-day hackfest*

![Figure 10 – Microsoft and Rivendel teams at the end of the 3-day Hackfest]({{ site.baseurl }}/images/2016-12-28-catraca-livre/Figure_10_Teams.jpg)

>“*With help from Microsoft and Rivendel, we mapped our development process and identified the main opportunities for improvement. Having A/B tests will significantly advance our capacity to deliver value to the company.*” —Alexandre de Maio, Catraca Livre

>“*We were very pleased to work with Microsoft and Catraca Livre on this project. This was an excellent opportunity for us to discover features we didn’t know Azure had. The platform is more versatile and powerful than we initially thought, and I was also pleased to find out that using the Azure API (through the Azure CLI program) allows for changes and actions which are not possible through the Azure Portal. This will benefit us in current and upcoming projects.*” —Diogo Lima, Rivendel

### Lessons learned ###

The hackfest was an opportunity for all involved to learn about technologies they were not familiar with. It was gratifying to observe that the adoption of certain DevOps disciplines (in this case, Infrastructure as Code and Release Management) together with some creative thinking can provide real business value, even for teams that are still evolving in their DevOps journey.

Some additional findings from the team are worth sharing:

- Sometimes resources created through the Azure API are not visible via the Azure Portal. It is important to become familiar with the Azure Resource Explorer (link 2 in the next section) which allows these resources to be seen and manipulated.
- Azure CLI is a very powerful tool. Developers and IT pros should not be restricted to only using the Azure Portal.
- Generalized and captured VHDs need to be stored in the same storage account where the VMs are (or where the VM Scale Set is).
- When using ARM templates, users should use parameters as much as possible. Leaving values hard-coded as strings can lead to issues when templates are reused, as conflicts can occur—and sometimes it is hard to find the source of these conflicts.

### Additional resources ###

1. [Authoring Azure Resource Manager templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates)
2. [Upload and create a Linux VM from custom disk image](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-linux-upload-vhd?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
3. [Azure Resource Explorer](http://resources.azure.com)
