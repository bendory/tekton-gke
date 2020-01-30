# Tekton and Workload Identity

## Advantages of Workload Identity

[GKE Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity) lets you associate a Kubernetes Service Account (KSA) with a Google IAM Service Account (GSA), such that any API request to a Google API will be authorized as that KSA/GSA. This makes it very easy to define Tekton workloads that authorize as Google Service Accounts, for instance to read from and write to Google Cloud Storage or Google Container Registry, deploy to Cloud Run or Kubernetes Engine, publish or subscribe to Cloud Pub/Sub topics, encrypt/decrypt data using Cloud KMS, and much more.

## Setting up Workload Identity

Create the Google IAM Service Account (GSA):

```
gcloud iam service-accounts create [GSA-NAME] \
    --display-name="Tekton Workload Identity"
```

Grant the account the `iam.workloadIdentityUser` role:

```
gcloud iam service-accounts add-iam-policy-binding \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:[PROJECT-ID].svc.id.goog[default/default]" \
  [GSA-NAME]@[PROJECT-ID].iam.gserviceaccount.com
```

Create the cluster with an `--identity-namespace`:

```
gcloud container clusters create [CLUSTER NAME] \
    --identity-namespace=[PROJECT-ID].svc.id.goog  \
```

Create the Kubernetes Service Account (KSA):

```
kubectl create serviceaccount [KSA-NAME]
```

**Note:** This creates the Service Account in the `default` namespace. To create it in another namespace, pass `--namespace=[NAMESPACE]`.

Create an IAM Policy Binding for the KSA:

```
gcloud iam service-accounts add-iam-policy-binding \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:[PROJECT-ID].svc.id.goog[default/[KSA-NAME]]" \
  [GSA-NAME]@[PROJECT-ID].iam.gserviceaccount.com
```

Annotate the KSA with its GSA information:

```
kubectl annotate serviceaccount \
  [KSA-NAME] \
  iam.gke.io/gcp-service-account=[GSA-NAME]@[PROJECT-ID].iam.gserviceaccount.com
```

## Run a TaskRun as the Service Account

Now that the GSA and KSA are created and linked, you can run a TaskRun as the KSA, which will authorize its requests to Google APIs as its linked GSA. You can check this by running a Task that invokes `gcloud auth list`:

```yaml
apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  name: auth-test
spec:
  steps:
  - image: google/cloud-sdk
    script: gcloud auth list
```

Run this Task as the KSA, using `tkn`:

```
$ tkn task start -f auth-test.yaml -s [KSA-NAME]
Taskrun started: auth-test-cb6ws
Waiting for logs to be available...
[unnamed-0]                    Credentialed Accounts
[unnamed-0] ACTIVE  ACCOUNT
[unnamed-0] *       [GSA-NAME]@[PROJECT-ID].iam.gserviceaccount.com
[unnamed-0] 
[unnamed-0] To set the active account, run:
[unnamed-0]     $ gcloud config set account `ACCOUNT`
```

This shows that `gcloud` was invoked with the credentials of your GSA, because it's linked to the specified KSA.

Now that we've demonstrated that `gcloud` can be invoked in Tekton  with service account credentials, you can extend it to interact with any GCP service you want.
