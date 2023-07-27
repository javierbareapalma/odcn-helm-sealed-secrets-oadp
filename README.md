This repo is intended to be used as a basic deployment for Red Hat OADP solution.

There are three 'Argocd' applications to be installed:
* app-of-apps
* operatorhub
* oadp-config

First application "app-of-apps" calls to deploy:
- *oadp-operator*
- *volsync-operator*

_oadp_ and _volsync_ operatos call in turn to the second application *operatorhub* templates to be installed.

For *seal-secrets-operator* this manifest might need these authorization below:
```
$ oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n sealed-secrets
$ oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller
```

Follow the order of OADP steps to be able to launch a backup.

For *03_dm-restic-secret-sealed*, some previous taks to be done:
```
$ kubeseal --controller-namespace=sealed-secrets  --fetch-cert > cert-ocp.pem
$ kubeseal --controller-namespace=sealed-secrets  <  dm-restic-secret.yaml --cert cert-ocp.pem > dm-restic-secret-sealed.yaml
```
Fetch the `dm-sealed-secret` CR-kind SealedSecret generated and put into _dmresticsecret_ value.
  
For *04_cloud-credentials-sealed*, some previous steps to be done:
* OBC Object Bucket Claim credentials and the bucket itself, will be required. The credentails are obtained from a secret after the creation of the bucket (OBC).
* In AWS environment, those credentials are (will be) the IAM creds.
```
$ oc -n openshift-adp create secret generic cloud-credentials --from-file cloud=s3-bucket-credentials.txt --dry-run=client -o yaml > cloud-credentials.yaml
$ kubeseal --controller-namespace=sealed-secrets  <  cloud-credentials.yaml --cert cert-ocp.pem > cloud-credentials-sealed.yaml
```
Fetch the `cloud-credentials` CR-kind SealedSecret generated and put into _cloudcreds_ value.

For *05_create-DPA-BSL*, where DPA=DataProtectionAgent and BSL=BackupStorageLocation, there will be three _values_oadp_X.yaml_ manifests to be chosen on behalf of the location of object repository.
- values_oadp_rgw.yaml
- values_oadp_aws.yaml
- values_oadp_noobaa.yaml

Accomodate the values-parameters accordingly beforehand.
