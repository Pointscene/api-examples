# Convert pointcloud to Potree structure and populate meta data
## Use labels in `resource.sync` task to identify resource later on
GraphQL
```
mutation {
  createWorkflow(
    name: "Create indexed pointcloud"
    instanceId: "{instanceId}"
    tasks: [
       {
        name: "input"
        type: "web.download"
        args: {
          url: "https://storage.googleapis.com/pointscene-sample-data/API-samples/pointcloud_L4133A4.laz"
        }
      }
      {
        name:"filter"
        type:"pdal.translate"
        inputs:["input"]
        args: {
          filters: [
            {
              voxeldownsize: {
                cell: 0.001
              }
            }
        	]
          writers: {
            las: {
              offsetX:"auto"
              offsetY:"auto"
              offsetZ:"auto"
            }
          }
      	}
      }
      {
        inputs: ["filter"]
        name: "potree"
        type: "potreeconverter.convert"
        args: {
          outputFormat:"LAS"
        }
      }
      {
        name: "info"
        type: "pdal.info"
        args: {}
        inputs: ["filter"]
      }
      {
        name: "sync"
        type: "resource.sync"
        args: {
          labels: {
            id:"indexed-pointcloud-1"
          }
        }
        inputs: ["info"]
      }
      {
        name: "output"
        type: "resource.upload"
        args: {}
        inputs: ["sync", "potree"]
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
-d '{"query": "mutation{createWorkflow(name:\"Create indexed pointcloud\" instanceId: \"{instanceId}\" tasks:[{name:\"input\" type:\"web.download\" args:{url:\"https://storage.googleapis.com/pointscene-sample-data/API-samples/pointcloud_L4133A4.laz\"}} {name:\"filter\" type:\"pdal.translate\" inputs:[\"input\"] args:{filters:[{voxeldownsize:{}}] writers:{las:{offsetX:\"auto\" offsetY:\"auto\" offsetZ:\"auto\"}}}}{inputs:[\"filter\"] name:\"potree\" type:\"potreeconverter.convert\" args:{outputFormat:\"LAS\"}}{name:\"info\" type:\"pdal.info\" args:{} inputs:[\"filter\"]}{name:\"sync\" type:\"resource.sync\" args:{labels:{id:\"indexed-pointcloud-1\"}} inputs:[\"info\"]}{name:\"output\" type:\"resource.upload\" args:{} inputs:[\"sync\", \"potree\"]}{name:\"enable\" type:\"resource.enable\" args:{} inputs:[\"sync\",\"output\"]}]){id}}"}' \
https://api.pointscene.com/graphql
```