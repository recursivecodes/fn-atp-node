# fn-atp-node-json

A sample Fn project that connects your function to an ATP database.

Download your ATP wallet via the UI or with the OCI CLI:
 
```bash
oci db autonomous-data-warehouse generate-wallet --autonomous-data-warehouse-id <atp db ocid> --password <new password for the truststore/keystore> --file /path/to/wallet.zip
``` 
 
Unzip it and rename the directory as `wallet`.  

Update `wallet/sqlnet.ora` to look like the following:

```bash
WALLET_LOCATION = (SOURCE = (METHOD = file) (METHOD_DATA = (DIRECTORY="/opt/oracle/instantclient_18_3/network/admin")))
SSL_SERVER_DN_MATCH=yes
```

Move `wallet/` it in the root of this project.

Create the table:

```sql
CREATE TABLE json_demo(
    id NUMBER GENERATED BY DEFAULT ON NULL AS IDENTITY,
    captured_at TIMESTAMP,
    data CLOB
    CONSTRAINT ensure_json CHECK (data IS JSON)
);

alter table json_demo add (
    constraint json_demo_pk PRIMARY KEY (id) 
);
```

Create app:

```bash
fn create app --annotation oracle.com/oci/subnetIds='["<subnet ocid>"]' --config DB_PASSWORD='<password>' --config DB_USER='<user>' --config CONNECT_STRING='<name of connection to use from tnsnames.>' fn-atp-node-json
```

Deploy functions:

```bash
fn deploy --app fn-atp-node-json --all
```

Invoke app:

```bash
echo '{"isAwesome": true, "isEasy": true}' | fn invoke fn-atp-node-json fn-atp-node-json-insert
fn invoke fn-atp-node-json fn-atp-node-json-read
```