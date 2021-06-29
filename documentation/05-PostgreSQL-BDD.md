[Retour menu principal](../README.md)

## 5. PostgreSQL BDD

The packaged PostgreSQL server can be configured to listen for TCP/IP connections, with the caveat that some non-critical scripts expect UNIX sockets and may misbehave.

In order to configure the use of TCP/IP for the database service, changes will need to be made to both ```postgresql``` and ```gitlab_rails``` sections of ```gitlab.rb```.

The following settings are affected in the postgresql block:

- ```listen_address``` controls the address on which PostgreSQL will listen.
- ```port``` controls the port on which PostgreSQL will listen, and must be set if ```listen_address is```.
- ```md5_auth_cidr_addresses``` is a list of CIDR address blocks which are allowed to connect to the server, after authentication via password.
- ```trust_auth_cidr_addresses``` is a list of CIDR address blocks which are allowed to connect to the server, without authentication of any kind. Be very careful with this setting. It is suggest that this be limited to the loopback address of 127.0.0.1/24 or even 127.0.0.1/32.
- ```sql_user``` controls the expected username for MD5 authentication. This defaults to ```gitlab```, and is not a required setting.
- ```sql_user_password``` sets the password that PostgrSQL will accept for MD5 authentication. Replace ```securesqlpassword``` in the example below with an acceptable password.

```yml
postgresql['listen_address'] = '0.0.0.0'
postgresql['port'] = 5432
postgresql['md5_auth_cidr_addresses'] = %w()
postgresql['trust_auth_cidr_addresses'] = %w(127.0.0.1/24)
postgresql['sql_user'] = "gitlab"
postgresql['sql_user_password'] = Digest::MD5.hexdigest "securesqlpassword" << postgresql['sql_user']
```

Any client or GitLab service which will connect over the network will need to provide the values of sql_user for the username, and password provided to the configuration when connecting to the PostgreSQL server. They must also be within the network block provided to md5_auth_cidr_addresses

To configure the gitlab-rails application to connect to the PostgreSQL database over the network, several settings must be configured.

- ```db_host``` needs to be set to the IP address of the database sever. If this is on the same instance as the PostgrSQL service, this can be 127.0.0.1 and will not require password authentication. **If you run postgresql in a Gitlab Omnibus instance, you need to leave this setting commented.
- ```db_port``` sets the port on the PostgreSQL server to connect to, and must be set if ```db_host``` is set.
- ```db_username``` configures the username with which to connect to PostgreSQL. This defaults to ```gitlab```.
- ```db_password``` must be provided if connecting to PostgreSQL over TCP/IP, and from an instance in the ```postgresql['md5_auth_cidr_addresses']``` block from settings above. This is not required if you are connecting to 127.0.0.1 and have configured ```postgresql['trust_auth_cidr_addresses']``` to include it.

```yml
gitlab_rails['db_host'] = '127.0.0.1'
gitlab_rails['db_port'] = 5432
gitlab_rails['db_username'] = "gitlab"
gitlab_rails['db_password'] = "securesqlpassword"
```

-----------------------------------------------------------------------------------------------------------------------------------

[Retour menu principal](../README.md)

[Suivant](06-Configurer-un-accès-LDAP.md)
