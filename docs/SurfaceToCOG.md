# Convert surface to cloud optimized surface
## Use labels in `resource.sync` task to identify resource later on
GraphQL
```
mutation {
  createWorkflow(
    instanceId: "{instanceId}"
    tasks: [
      {
        name: "input"
        type: "web.download"
        args: {
          url: "https://storage.googleapis.com/pointscene-sample-data/API-samples/pointcloud_L4133A4.tif"
        }
      }
      {
        inputs: ["input"]
        name: "cog"
        type: "gdal.translate"
        args: {
          of: "COG"
          co: ["COMPRESS=LZW"]
      	}
      }
      {
        name: "info"
        type: "gdal.info"
        args: {}
        inputs: ["cog"]
      }
      {
        name: "sync"
        type: "resource.sync"
        args: {
          labels: {
            id:"surface-cog-1"
          }
        }
        inputs: ["info"]
      }
      {
        name: "output"
        type: "resource.upload"
        args: {}
        inputs: ["sync", "cog"]
      }
      {
        name: "enable"
        type: "resource.enable"
        args: {}
        inputs: ["sync", "output"]
      }
    ]
  ) {
    id
  }
}
```

curl
```
curl \
-X POST \
-H "Content-Type: application/json" \
-H "Authorization: Bearer {TOKEN}" \
-d '{"query": "mutation{createWorkflow(instanceId:\"{instanceId}\"tasks:[{name:\"input\" type:\"web.download\" args:{url:\"https://storage.googleapis.com/pointscene-sample-data/API-samples/pointcloud_L4133A4.tif\"}}{inputs:[\"input\"] name:\"cog\" type:\"gdal.translate\" args:{of:\"COG\" co:[\"COMPRESS=LZW\"]}}{name:\"info\" type:\"gdal.info\" args:{} inputs:[\"cog\"]}{name:\"sync\" type:\"resource.sync\" args:{labels:{id:\"surface-cog-1\"}} inputs:[\"info\"]}{name:\"output\" type:\"resource.upload\" args:{} inputs:[\"sync\", \"cog\"]}{name:\"enable\" type:\"resource.enable\" args:{} inputs:[\"sync\", \"output\"]}]){id}}"}' \
https://api.pointscene.com/graphql
```