# Welcome to the OpenShift Virtualization Roadshow extra exercises


## AAP on OpenShift

Fork this repo first!

### Installing

1. Install the AAP operator and add a controller: `oc apply -f manifests/aap-operator-install.yaml`
2. Get the route for the controller: `oc get routes -n ansible-automation-platform controller`
3. Get the login for the controller: `oc extract -n ansible-automation-platform secret/controller-admin-password --to -`
4. Login to the controller, request a trial subscription.

### Setting up a dynamic Kubevirt Inventory in the AAP Controller

First we need a credential. This token *should be scoped* but for now let's just add a cluster-admin scoped token.

1. Add a ServiceAccount: `oc create sa controller-credential`
2. Add cluster-admin rights to the service account: `oc adm policy add-cluster-role-to-user cluster-admin -z controller-credential` 
3. Create a token for the SA: `oc create token controller-credential --duration=4294967296s` (hmm 136 years, 29 weeks, 3 days, 6 hours, 28 minutes, 16 seconds. :-0). Copy it and add it as a Credential in the controller:
![alt text](image.png) It's also possible to add the Credential using the a declarative approach. Apply a `kind: AnsibleCredential` to your cluster. See the example [here.](https://github.com/ansible/awx-resource-operator/blob/devel/config/samples/credentials/tower_v1alpha1_ansiblecredential-bearer.yaml).

4. Connect the Resource operator to the automation controller:

    a. In the navigation panel, select Access Users.

    b. Select the username you want to create a token for.

    c. Click on Tokens, then click Add. Copy the token.

    d. You can leave Applications empty. Add a description and select Read or Write for the Scope.

    Add a secret with the token:

    ```bash
    oc apply -f - <<EOF
    apiVersion: v1
    kind: Secret
    metadata:
      name: controller-access
    type: Opaque
    stringData:
      token: <generated-token>
      host: https://<my-controller-host.example.com>
    EOF
    ```

5. Create a new project:

    ```bash
    oc apply -f - <<EOF
    apiVersion: tower.ansible.com/v1alpha1
    kind: AnsibleProject
    metadata:
      name: ocp-virt
    spec:
      repo: https://github.com/cldmnky/ocp-virt-roadshow
      branch: main
      name: ocp-virt
      scm_type: git
      organization: Default
      description: 'OCP Virt Lab' 
      connection_secret: controller-access
      runner_pull_policy: IfNotPresent
    EOF
    ```

