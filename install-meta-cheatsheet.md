# install-meta test cheat-sheet

## The fix needed (Monday)

In `vdi-lib-plugin-eda/bin/install-meta`, `validate_variables` and `update_metadata`
both use `stable_id` to look up and update rows in the `attributegraph` table.
They should use `provider_label` instead — that's where the column names (`value`, `lat`, `long`) live.

## SSH tunnel

```bash
ssh -L 5433:ares13.penn.apidb.org:5432 cedar -N &
```

Test from host (or container with host networking):
```bash
bash -c 'echo >/dev/tcp/localhost/5433' && echo "open" || echo "closed"
```

## docker-compose.override.yml

`network_mode: host` is already added. The `./bin` mount is commented out so
`install-meta` (copied from `vdi-lib-plugin-eda` during build) is visible.

## Env vars inside the container

```bash
export DB_HOST=localhost
export DB_PORT=5433
export DB_NAME=genomicsdb_069n
export DB_SCHEMA=vdi_datasets_dev_n
export DB_USER=<user>
export DB_PASS=<pass>
```

## Database

- LDAP CN: `genomicsdb_devn` (PostgresContext)
- Host: `ares13.penn.apidb.org:5432` (reach via tunnel above)
- DB name: `genomicsdb_069n`
- Schema: `vdi_datasets_dev_n`
- Not in LDAP as `vdi*` or `eda*` — lives under `genomicsdb_devn`

## Test annotations file

```bash
mkdir /tmp/propsdir
cat > /tmp/propsdir/annotations.tsv << 'EOF'
variable	label	definition
value	Value	A numeric measurement value
lat	Latitude	Geographic latitude in decimal degrees
long	Longitude	Geographic longitude in decimal degrees
EOF
```

## Running install-meta

```bash
install-meta <user_dataset_id> /dev/null /tmp/propsdir
```

`meta.json` is ignored by the script so `/dev/null` is fine.

Test dataset user_dataset_id: `ccA5wYMN1l1ws`


## docker-compose.override.yml for vdi-plugin-wrangler 


network_mode: host is important
(we can probably mount the install-meta script in to another location to save rebuilding etc)




services:
  plugin:
    network_mode: host
    volumes:
#      - ./bin:/opt/veupathdb/bin  # Mount local bin directory
      - ./lib/R:/opt/veupathdb/lib/R  # Mount local R code
      - ./tests:/opt/veupathdb/tests # and the test data
      - ./config/local-dev-config.yml:/etc/vdi/config.yml  # Mount VDI server config
