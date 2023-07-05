# GraphQL Troubleshooting

Some possible error cases

## Error 1 - `FetchError: request to http://my.service.com/graphql failed` 

### Solution 1

Use one of the available schema urls, not literally the placeholder value `my.service.com`

```
glass apollo download-schema -p Registry --schema "http://ce-gateway.ce-orchestration.k8s.prod.walmart.com/graphql/"
```

## Error 2 - `failed, reason: self signed certificate in certificate chain`

### Solution 2

Schema url in script should be `http` not `https`
