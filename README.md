# Dynamic-Secrets-Database-Secrets-Engine-HashiCorp-Vault Postgres Database

## We use initial_token, but in production you must use auth method, policies with capabilities and so on
#### We use EC2 Instance with IAM Roles( RDSFullAccess) , and Postgres, which is deployed on RDS 

![assets](https://user-images.githubusercontent.com/73755890/202867199-5ac7aa88-e6d2-43ec-9679-47a5a2e57e5e.png)

first of all we configure Vault on Amazon Linux, configuration file here:

###### vault_on_amazon_linux



![assets](https://user-images.githubusercontent.com/73755890/202867387-342f0026-7ffc-4346-abc5-56814a5d2f2f.png)


### Postgres Basic Configuration
Configuration file here:
##### postgres_basic

#### We configure schema in test_db, if we don't connect to db, we create our schema on default database postgres

### Enable Secret Engine

> vault secrets enable -path=psql database

> vault write psql/config/testDB \
    plugin_name=postgresql-database-plugin \
    allowed_roles="developer-role" \
    connection_url="postgresql://{{username}}:{{password}}@database-1.xxxxxxxxx.rds.database:5432/test_db?sslmode=disable" \
    username="postgres" \
    password="postgres"
   
##### username and password dynamic

### We can check it with:
> vault read psql/config/testDB

`---                                   -----`
`allowed_roles                         [developer-role]`
`connection_details                    map[connection_url:postgresql://{{username}}:{{password}}@database-1.xxxxxxxxxxxxxxx:5432/test_db?sslmode=disable username:postgres]`
`plugin_name                           postgresql-database-plugin`
`root_credentials_rotate_statements    []`

### Now we create a role for this database

> vault write psql/roles/developer-role \
    db_name=testDB \
    creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
    GRANT ALL PRIVILEGES ON DATABASE test_db TO \"{{name}}\"; \
    GRANT ALL PRIVILEGES ON SCHEMA testms TO \"{{name}}\";"\
    default_ttl="1h" \
    max_ttl="24h"
    
 #### it's example and you can change it for example for individual access without shemas, without all privileges and so on.
 
 ![1576778435-vault-db](https://user-images.githubusercontent.com/73755890/202867762-611e79b2-4832-4f0e-b6af-a6b0df0052e9.png)

 
 ### After that we can retrieve a username and password 
 
 > vault read psql/creds/developer-role

`Key                Value`
`---                -----`
`lease_id           psql/creds/developer-role/fyF5xDomnKeCHNZNQgStwBKD`
`lease_duration     1h`
`lease_renewable    true`
`password           A1a-ckirtymYaXACpIHn`
`username           v-token-readonly-6iRIcGv8tLpu816oblPY-1556567086`

 
 
 ### We can check it 
 
 > psql -h database-1.xxxxxxxxxxxxxxxxx.rds.database \
    -d test_db \
    -U v-root-develope-N2QYxWSbvc5ZFduq6JyY-1615063136 \
    -W
Password: ZOhXBsoZcb6uz5ACFge-


### For Vault list ID

> vault list sys/leases/lookup/psql/creds/developer-role

### Renew lease
> vault lease renew psql/creds/developer-role/vmzoyrTu9PAPKWfTu5qGPqdc

### Revoke ID

> vault lease revoke psql/creds/developer-role/vmzoyrTu9PAPKWfTu5qGPqdc

### Revoke All IDs

> vault lease revoke -prefix psql/creds/developer-role

### Retrive username and password via API

> curl --header "X-Vault-Token: xxxxxxxxxxxxxxxxxxxxx" \
      --request GET \
      http://xxxxxxxxxxxxxxxx:8200/v1/psql/creds/developer-role | jq
      
      
![Untitled](https://user-images.githubusercontent.com/73755890/202868291-b31e8027-ed87-4cb2-a363-7494f1d9adf4.png)

    [Link to PostgreSQL Database Secret Engine Documentation](https://developer.hashicorp.com/vault/docs/secrets/databases/postgresql)
    [Link to PosgreSQL Documentation](https://www.postgresql.org/docs/current/index.html)


##### THAT'S ALL. THANKS!
      
