---
layout: post
title:  "Setting up EnMasse on AWS EC2"
author: Ulf Lilleengen
categories: technical enmasse messaging kubernetes aws
---

As I was preparing a demo for my [presentation](http://rivieradev.fr/session/130) at the [RivieraDev](http://rivieradev.fr/) next week, I wrote a guide for setting up EnMasse on AWS in the same way as I am doing for the demo. This is not even very specific to AWS, so you can probably modify the configuration to fit Microsoft Azure or even Google GCE. 

The end result from this guide is an instance of EnMasse suitable for development and/or experimentation, and should not be considered a production ready setup. For instance, no persistence is configured, so neither messages in brokers nor state in other components like hawkular are persisted.

## Prerequisites

First, you must have created an [EC2 instance](https://aws.amazon.com/ec2/). EnMasse runs on OpenShift and Kubernetes, but this post uses OpenShift purely for convenience. Have a look at the [OpenShift prerequisites](https://docs.openshift.org/latest/install_config/install/prerequisites.html) for the required hardware configuration. The installation will be done using [Ansible](https://www.ansible.com), so make sure Ansible is installed on laptop or workstation.

### Configure Ansible to handle passwordless sudo

For EC2 instance, the default is a passwordless sudo, and Ansible (2.3.0.0 at the time of writing) requires a minor configuration modification to deal with that.  On the host you will be running ansible from, edit /etc/ansible/ansible.cfg, and make sure that the `sudo_flags` parameter is set to `-H -S` (remove the `-n`).

## Setting up OpenShift

Once Ansible is setup, installing OpenShift is easy. First, an inventory file with the configuration
and the hosts must be created. Save the following configuration to a file, i.e. `ansible-inventory.txt`:

    [OSEv3:children]
    masters
    nodes

    [OSEv3:vars]
    deployment_type=origin
    openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]
    openshift_master_default_subdomain=<yourdomain>
    openshift_public_hostname=openshift.<yourdomain>
    openshift_hostname=<ec2 instance hostname>
    openshift_metrics_hawkular_hostname=hawkular-metrics.<yourdomain>

    openshift_install_examples=false
    openshift_hosted_metrics_deploy=true

    [masters]
    <ec2 host> openshift_scheduleable=true openshift_node_labels="{'region': 'infra'}"

    [nodes]
    <ec2 host> openshift_scheduleable=true openshift_node_labels="{'region': 'infra'}"

This will configure OpenShift so that it can only be accessed by users defined in `/etc/origin/master/htpasswd`.

If you don't have a domain with wildcard support, you can replace <yourdomain> with <ip>.nip.io, and
you will have a working setup without having a specialized domain. 

You can now download the ansible playbooks. The simplest way to do this is to just clone the git
repository:

    git clone https://github.com/openshift/openshift-ansible.git

To install OpenShift, run the playbook like this

    ansible-playbook -u ec2-user -b --private-key=<keyfile>.pem -i ansible-inventory.txt openshift-ansible/playbooks/byo/openshift-cluster/config.yml

This command will take a while to finish.

### Creating a user

To be able to deploy EnMasse in OpenShift, a user must be created. Log on to your EC2
instance, and create the user:

    htpasswd -c /etc/origin/master/htpasswd <myuser>

Where `<myuser>` is the username you want to use. The command will prompt you for a password that
you will later use when deploying EnMasse.

## Creating certificates

To be able to access your EnMasse cluster outside OpenShift, you must create a certificate for it.
For testing purposes, you can create a self-signed key and certificate like this:

    openssl req -new -x509 -batch -nodes -out server-cert.pem -keyout server-key.pem

## Setting up EnMasse

You can find the latest version of EnMasse [here](https://github.com/EnMasseProject/enmasse/releases/latest). To deploy EnMasse, it is recommended to use the deploy script together with a template of the latest version. At the time of writing, the latest version is 0.9.0, which can be deployed as follows:

    curl -L https://github.com/EnMasseProject/enmasse/releases/download/0.9.0/enmasse-deploy.sh -o enmasse-deploy.sh
    bash enmasse-deploy.sh -c https://openshift.<yourdomain>:8443 -p enmasse -t https://github.com/EnMasseProject/enmasse/releases/download/0.9.0/enmasse-template.yaml -u <myuser> -k server-key.pem -s server-cert.pem -p enmasse

Now you have EnMasse deployed and ready to use. The messaging endpoint will be messaging-enmasse.<yourdomain> for AMQP, mqtt-enmasse.<yourdomain> for MQTT, and the console will be available at console-enmasse.<yourdomain>, where you can create and delete addresses and monitor your system.

## (Optional) Setting up metrics

The process for setting up grafana is a bit more involved, but it gives you a nice overview of whats
going on over time. First of all, I like to setup everything metric-related in the `openshift-infra`
project. To do that, you must first give your user permission sufficient privileges. In this setup,
since it's not a production setup, I grant cluster-admin privileges for simplicity (requires logging
into the ec2 instance):

    oadm --config /etc/origin/master/admin.kubeconfig policy add-cluster-role-to-user cluster-admin developer

With this in place, you can setup the [hawkular-openshift-agent](https://github.com/hawkular/hawkular-openshift-agent) which pulls metrics from routers and brokers:

    oc create -f https://raw.githubusercontent.com/openshift/origin-metrics/master/hawkular-agent/hawkular-openshift-agent-configmap.yaml -n openshift-infra
    oc process -f https://raw.githubusercontent.com/openshift/origin-metrics/master/hawkular-agent/hawkular-openshift-agent.yaml IMAGE_VERSION=1.4.0.Final | oc create -n openshift-infra -f -
    oc adm policy add-cluster-role-to-user hawkular-openshift-agent system:serviceaccount:openshift-infra:hawkular-openshift-agent

If everything is setup correctly, you can then deploy [Grafana](https://grafana.com/):

    oc process -f https://raw.githubusercontent.com/hawkular/hawkular-grafana-datasource/master/docker/openshift/openshift-template-ephemeral.yaml -n openshift-infra | oc create -n openshift-infra -f -

After some time, Grafana should become available at `oc get route -n openshift-infra -o jsonpath='{.spec.host}' hawkular-grafana`. The default username and password is `admin/admin`.

## Summary

In this post, you've seen how to:

    * Deploy OpenShift on an AWS EC2 instance
    * Deploy EnMasse cloud messaging
    * Deploy Grafana for monitoring

If you have questions regarding the setup, don't hesitate to get in touch on
[twitter](https://twitter.com/lulf), e-mail me directly, or post on the
[EnMasse mailing list](https://www.redhat.com/mailman/listinfo/enmasse).
