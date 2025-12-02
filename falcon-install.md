# Environment Variables

```bash
export CID=""    
export FALCON_CLIENT_ID=""  
export FALCON_CLIENT_SECRET=""  
export CLUSTER_NAME=""  
export SENSOR_TOKEN=""  
export SENSOR_REPO=""  
export SENSOR_TAG=""  
export IAR_TOKEN=""  
export IAR_REPO=""  
export IAR_TAG=""  
export KAC_TOKEN=""  
export KAC_REPO=""  
export KAC_TAG=""
```

# Step 1 - Complete Prerequisites 

### Ensure workstation / cloudshell has the following installed

curl
helm
kubectl

## Download the sensor download script and make it executable

```bash
curl https://raw.githubusercontent.com/CrowdStrike/falcon-scripts/refs/heads/main/bash/containers/falcon-container-sensor-pull/falcon-container-sensor-pull.sh -o falcon-container-sensor-pull.sh

chmod 777 falcon-container-sensor-pull.sh
```

## Get CID Value from Sensor Download Page 

1. Log into Falcon and copy CID value from Sensor Download page. (Host Management and Setup -> Sensor Download)
2. Paste CID Value into variable CID.

## Generate API Client for Sensor Install 

1. CrowdStrike API Client is needed with the following scopes:

Falcon Images Download (read)  
Sensor Download (read)  
Falcon Container CLI (write)  
Falcon Container Image (read/write)  

2. Paste the Client ID into the variable FALCON_CLIENT_ID 
3. Paste the Secret into the variable FALCON_CLIENT_SECRET

## Setup Node Affinity Falcon Sensor to specific Nodes (Optional)

1. Label specific nodes, replace node1 node2 node3 values with node name values

```bash
kubectl label nodes node1 node2 node3 falcon-sensor=enabled
```

2. Create a custom-values.yaml file:
```bash
  daemonset:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: falcon-sensor
            operator: In
            values:
            - enabled
```


# Step 2 - Use Pull Script to Gather Image Paths and Pull Tokens for Sensor, IAR, KAC.

## Fetch Pull Token for Sensor 

1. Use the pull script to get the pull token for the Falcon sensor

```bash
./falcon-container-sensor-pull.sh \
--client-id "$FALCON_CLIENT_ID" \
--client-secret "$FALCON_CLIENT_SECRET" \
--type falcon-sensor \
--get-pull-token
```
2. Paste the output into the variable SENSOR_TOKEN

## Pull Image Path for Sensor

1. Use the pull script to get the image path for the Falcon sensor

```bash
./falcon-container-sensor-pull.sh \
--client-id "$FALCON_CLIENT_ID" \
--client-secret "$FALCON_CLIENT_SECRET" \
--type falcon-sensor \
--get-image-path
```
2. Paste the image path into the variable SENSOR_REPO (This is everything before the ":")
3. Paste the image tag into the variable SENSOR_TAG (This is everything after the ":")

## Fetch Pull Token for IAR 

1. Use the pull script to get the pull token for the IAR

```bash
./falcon-container-sensor-pull.sh \
--client-id "$FALCON_CLIENT_ID" \
--client-secret "$FALCON_CLIENT_SECRET" \
--type falcon-imageanalyzer \
--get-pull-token
```

2. Paste the output into the variable IAR_TOKEN

## Pull Image Path for IAR

1. Use the pull script to get the image path for the IAR

```bash
./falcon-container-sensor-pull.sh \
--client-id <FALCON_CLIENT_ID> \
--client-secret <FALCON_CLIENT_SECRET> \
--type falcon-imageanalyzer \
--get-image-path
```

2. Paste the image path into the variable IAR_REPO (This is everything before the ":")
3. Paste the image tag into the variable IAR_TAG (This is everything after the ":")

## Fetch Pull Token for KAC 

1. Use the pull script to get the pull token for the KAC

```bash
./falcon-container-sensor-pull.sh \
--client-id "$FALCON_CLIENT_ID" \
--client-secret "$FALCON_CLIENT_SECRET" \
--type falcon-imageanalyzer \
--get-pull-token
```

2. Paste the output into the variable KAC_TOKEN

## Pull Image Path for KAC

1. Use the pull script to get the image path for the KAC

```bash
./falcon-container-sensor-pull.sh \
--client-id <FALCON_CLIENT_ID> \
--client-secret <FALCON_CLIENT_SECRET> \
--type falcon-imageanalyzer \
--get-image-path
```

2. Paste the image path into the variable KAC_REPO (This is everything before the ":")
3. Paste the image tag into the variable KAC_TAG (This is everything after the ":")

# Step 3 - Use Helm to install Sensor, IAR, KAC

## Helm Install for Sensor (Deployed to ALL Nodes)

```bash
helm repo add crowdstrike https://crowdstrike.github.io/falcon-helm --force-update
helm upgrade --install falcon-sensor crowdstrike/falcon-sensor -n falcon-system --create-namespace \
--set falcon.cid="$CID" \
--set node.image.repository="$SENSOR_REPO" \
--set node.image.tag="$SENSOR_TAG" \
--set node.image.registryConfigJSON="$SENSOR_TOKEN"
```

## Helm Install for Sensor (Deployed to labeled Nodes- OPTIONAL)

```bash
helm repo add crowdstrike https://crowdstrike.github.io/falcon-helm --force-update
helm upgrade --install falcon-sensor crowdstrike/falcon-sensor -n falcon-system --create-namespace \
--values custom-values.yaml \
--set falcon.cid="$CID" \
--set node.image.repository="$SENSOR_REPO" \
--set node.image.tag="$SENSOR_TAG" \
--set node.image.registryConfigJSON="$SENSOR_TOKEN"
```

## Helm Install for IAR ###

```bash
helm upgrade --install iar crowdstrike/falcon-image-analyzer \
  -n falcon-image-analyzer --create-namespace \
  --set deployment.enabled=true \
  --set crowdstrikeConfig.cid="$CID" \
  --set crowdstrikeConfig.clientID="$FALCON_CLIENT ID" \
  --set crowdstrikeConfig.clientSecret="$FALCON_CLIENT_SECRET" \
  --set image.repository="$IAR_REPO" \
  --set image.tag="$IAR_TAG" \
  --set crowdstrikeConfig.clusterName="$CLUSTER_NAME" \
  --set image.registryConfigJSON="$IAR_TOKEN"
```

## Helm Install for KAC ###

```bash
helm repo add crowdstrike https://crowdstrike.github.io/falcon-helm --force-update
helm upgrade --install falcon-kac crowdstrike/falcon-kac -n falcon-kac --create-namespace \
--set falcon.cid="$CID" \
--set image.repository="$KAC_REPO" \
--set image.tag="$KAC_TAG" \
--set image.registryConfigJSON="$KAC_TOKEN"
```

