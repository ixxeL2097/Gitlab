[Retour menu principal](../README.md)

## 8. Grafana

Grafana is a powerful dashboard building system that you can use to visualize performance metrics from the embedded Prometheus monitoring system.

Starting with GitLab 12.0, Grafana is enabled by default and SSO with GitLab is automatically configured. Grafana will be available on ```https://gitlab.example.com/-/grafana```

Logging in to Grafana using username/password combo is disabled , and only GitLab SSO is available by default. However, to access the admin account, you need to enable login using username/password. For that, add the following line to ```/etc/gitlab/gitlab.rb``` file and reconfigure:
```yml
grafana['disable_login_form'] = false
```
Configure admin password:
```yml
grafana['admin_password'] = 'foobar'
```
**If no admin password is provided, Omnibus GitLab will automatically generate a random password for the admin user as a security measure. However, in that case you will have to reset the password manually to access the admin user.**

Starting with GitLab 11.10, dashboards for monitoring Omnibus GitLab will be pre-loaded and available on initial login.
For earlier versions of GitLab, you can manually import the pre-built dashboards that are tailored for Omnibus installations

Documentation for Gitlab Grafana is available at ```https://docs.gitlab.com/omnibus/settings/grafana.html```

-----------------------------------------------------------------------------------------------------------------------------------

[Retour menu principal](../README.md)

[Suivant](09-Backup.md)
