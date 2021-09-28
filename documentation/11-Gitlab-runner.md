[Retour menu principal](../README.md)

## 11. Gitlab runner

### Kubernetes installation using Helm

Installation documentation : 

- https://docs.gitlab.com/runner/install/kubernetes.html

First add the helm repo and download the chart :

```shell
helm repo add gitlab https://charts.gitlab.io
helm fetch --untar gitlab/gitlab-runner
```

Once the chart is downloaded, you can inject your Gitlab server URL and runner token in the `values.yaml` file. The runner token can be a shared one or a specific one. Specific runner are created from a project in Gitlab and shared runner are created from the admin area.

<p align="center">
  <img src="../pictures/gitlab-runner-token.png" width="80%" height="80%">
</p>

values.yaml :

```yaml
## The GitLab Server URL (with protocol) that want to register the runner against
## ref: https://docs.gitlab.com/runner/commands/README.html#gitlab-runner-register
##
gitlabUrl: https://gitlab-backup.devibm.local:2443/

## The Registration Token for adding new Runners to the GitLab Server. This must
## be retrieved from your GitLab Instance.
## ref: https://docs.gitlab.com/ce/ci/runners/README.html
##
runnerRegistrationToken: "t1SzL9itEugqmmxtqhJi"
```

If you have a TLS secured Gitlab server, you will need to upload the certificate to the runner. First fetch the certificate from the server : 

```shell
openssl s_client -showcerts -connect gitlab-backup.devibm.local:2443 | openssl x509 -outform PEM > gitlab-backup.pem
```

Then create a kubernetes secret with the fetched certificate file. The secret must be created in the same namespace as the gitlab runner is installed :

```shell
kubectl create secret generic gitlab-backup-cert --namespace gitlab-runner --from-file=gitlab-backup.crt
```

And finally update your `values.yaml` to take into account this secret :

```yaml
## Set the certsSecretName in order to pass custom certficates for GitLab Runner to use
## Provide resource name for a Kubernetes Secret Object in the same namespace,
## this is used to populate the /home/gitlab-runner/.gitlab-runner/certs/ directory
## ref: https://docs.gitlab.com/runner/configuration/tls-self-signed.html#supported-options-for-self-signed-certificates
##
certsSecretName: "gitlab-backup-cert"
```

You can now install your gitlab runner in your kubernetes cluster :

```
helm upgrade -i runner --namespace gitlab-runner gitlab-runner/
```

If you run into a `x509 certificate unknow authority` error with your gitlab runner container, you might need to use a slight workaround. You can modify the 
`/templates/_env_vars.tpl` file from the gitlab runner helm chart to add following lines in the `{{- define "gitlab-runner.runner-env-vars" }}` section :

```tpl
- name: CI_SERVER_TLS_CA_FILE
  value: /home/gitlab-runner/.gitlab-runner/certs/gitlab-backup.crt
```

Then reinstall the gitlab runner instance and everything should be working properly this time.
