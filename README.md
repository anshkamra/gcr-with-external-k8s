# gcr-with-external-k8s
#If you run Kubernetes on Google Cloud Platform (GCP), Google Cloud Registry (GCR) support should be automatic. You must still make sure, however, that your instance has the correct permissions to access your private images. For details, see Using Container Registry with Google Cloud Platform. To connect to GCR from an environment other than GCP, you add an ImagePullSecrets field to the configuration for a Kubernetes service account. This is a type of Kubernetes secret that contains credential information.


# create a GCP service account; format of account is email address
SA_EMAIL=$(gcloud iam service-accounts --format='value(email)' create k8s-gcr-auth-ro)

# create the json key file and associate it with the service account
gcloud iam service-accounts keys create k8s-gcr-auth-ro.json --iam-account=$SA_EMAIL

# get the project id
PROJECT=$(gcloud config list core/project --format='value(core.project)')

# add the IAM policy binding for the defined project and service account
gcloud projects add-iam-policy-binding $PROJECT --member serviceAccount:$SA_EMAIL --role roles/storage.objectViewer



# create the secret and specify the file that you just created

SECRETNAME=varSecretName

kubectl create secret docker-registry $SECRETNAME \
  --docker-server=https://gcr.io \
  --docker-username=_json_key \
  --docker-email=user@example.com \
  --docker-password="$(cat k8s-gcr-auth-ro.json)"


# add the secret to your Kubernetes configuration

SECRETNAME=varSecretName
kubectl patch serviceaccount default \
  -p "{\"imagePullSecrets\": [{\"name\": \"$SECRETNAME\"}]}"


# add your imagePullSecrets value, the YAML for the default service account looks like the following

apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
  namespace: default
  selfLink: /api/v1/namespaces/default/serviceaccounts/default
imagePullSecrets:
  - name: your_secret_name


# Or you can specify the imagePullSecrets value on individuals pods:

