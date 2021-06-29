[Retour menu principal](../README.md)

## 6. Configure LDAP access

Documentation for Gitlab LDAP configuration is available here:

- https://docs.gitlab.com/ee/administration/auth/how_to_configure_ldap_gitlab_ce/index.html
- https://docs.gitlab.com/ee/administration/auth/how_to_configure_ldap_gitlab_ee/index.html
- https://docs.gitlab.com/omnibus/settings/ldap.html
- https://docs.gitlab.com/ee/administration/auth/ldap.html
- https://docs.gitlab.com/ee/administration/auth/ldap-ee.html

To configure LDAP access, you need to add a dedicated account to your AD server and **allow logon for this user**. This user will perform LDAP search through the AD DB to check for users allowed to connect to Gitlab.

For docker-compose, you need to add the following lines to your file:

```yml
environment:
  GITLAB_OMNIBUS_CONFIG: |
        
    ## LDAP
    gitlab_rails['ldap_enabled'] = true
    gitlab_rails['prevent_ldap_sign_in'] = false
    gitlab_rails['ldap_servers'] = YAML.load <<-EOS
      main:
        label: 'YOUR-CUSTOME-NAME'
        host: '${LDAP_HOSTNAME}'                # Your AD server hostname
        port: 636                               # 389 or 636 (for secure)
        uid: 'sAMAccountName'
        encryption: 'simple_tls'                # "start_tls" or "simple_tls" or "plain"
        verify_certificates: false  
        bind_dn: '${LDAP_BIND_DN}'              # DN of the user created to LDAP search your AD server
        password: '${LDAP_BIND_PASSWD}'         # password for user
        active_directory: true                  # true if Microsoft AD
        allow_username_or_email_login: true
        base: '${LDAP_BASE_OU}'                 # OU in which Gitlab will search for users that can access Gitlab
    EOS
```
_P.S: please stick closely to the indentation in order not to have syntax errors_


The DN of the ```bind_dn``` parameter must be properly filled as per below:
```console
ex: CN=gitlab_ldap,OU=Gitlab_users,DC=example,DC=com
```
The ```base``` parameter must be an OU in which users who will be allowed to sign in to gitlab are. For example:
```console
ex: OU=Gitlab,DC=example,DC=com
```

Once deployed, you can easily check working LDAP connection with following commands:
```console
[root@gitlabserver ~]# docker exec -it gitlab bash
root@gitlabserver:/# gitlab-rake gitlab:ldap:check
Checking LDAP ...

LDAP: ... Server: ldapmain
not verifying SSL hostname of LDAPS server 'srv-ad1.example.com:636'
LDAP authentication... Success
LDAP users with access to your GitLab server (only showing the first 100 results)
        DN: cn=gitlabusers,ou=gitlabgroups,ou=gitlab,dc=example,dc=com      sAMAccountName: GitlabUsers
        DN: cn=gitlabadmins,ou=gitlabgroups,ou=gitlab,dc=example,dc=com     sAMAccountName: GitlabAdmins

Checking LDAP ... Finished
```

Gitlab allows LDAP group synchronization in **Enterprise version (gitlab-ee) only**. You can specify some groups with specific access to Gitlab projects, and you can also specify a unique group which has top level gitlab administrator access.

You can do that by adding the following parameters in your Gitlab LDAP configuration (after ```base``` parameter):

```yml
group_base: '${LDAP_BASE_GROUP}'
admin_group: '${LDAP_BASE_ADMIN}'  
```
The parameter ```group_base``` must be an OU in which your Gitlab groups are located. For example:

```console
ex: OU=groups,OU=Gitlab_users,DC=example,DC=com
```

The parameter ```admin_group``` must be the CN of the group you want to grant admin privileges. DO NOT specify a full DN. For example:

```console
ex: GitlabAdmins
```

Once everything is configured, you have to log in a first time with some users inside the ```base``` OU parameter in order for gitlab to be aware of those users.
Then you can log in to gitlab and configure access for these users.

if for some reason, your want to force synchronization you can always execute following command inside gitlab container:

```bash
gitlab-rails runner 'LdapGroupSyncWorker.perform_async'
```

It will help for example to refresh admin group list inside gitlab.

You can see documentation about it on this link:

- https://docs.gitlab.com/ee/administration/auth/how_to_configure_ldap_gitlab_ee/#group-sync


-----------------------------------------------------------------------------------------------------------------------------------

[Retour menu principal](../README.md)


[Suivant](07-Prometheus.md)
