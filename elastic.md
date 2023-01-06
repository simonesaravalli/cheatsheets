# Elastic Stack utils

## Unlock Elastic read only indexes:

```
curl -x PUT https://<elastic url>/_all/_settings -H 'Content-Type: application/json' -d'{ "index.blocks.read_only_allow_delete" : null } }'
```