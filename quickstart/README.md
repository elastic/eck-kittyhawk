# Quickstart

## Base Diagram

```mermaid
flowchart LR
	req(Data Sources) -.-> req3
	req(Data Sources) -.-> req2
	req2(Elastic Agent) ---> |System, Kubernetes Integrations|Elasticsearch_Cluster
	req3(Elastic Agent) ---> |System, Kubernetes Integrations|Elasticsearch_Cluster
	req4(Fleet Server) --- Elasticsearch_Cluster
	subgraph Elasticsearch_Cluster
	es1 --- es2
	es2 --- es3
	es3 --- es1
	end
	Elasticsearch_Cluster --- Kibana
	Elasticsearch_Cluster --- A[/Kubernetes Secret: License\]
```


## Deployable Assets in this Directory

| Elastic Cluster | Filename | Resource (Kind) | Count |  Features Added |
| :-------------: |:-------------:| :-------------: | :-------------: | :-------------: |
|main|elasticsearch.yml|Elasticsearch|3|[Virtual Memory](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/virtual-memory), [Internal Monitoring](https://www.elastic.co/docs/deploy-manage/monitor/stack-monitoring/eck-stack-monitoring), [Ephemeral Drives](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/volume-claim-templates#k8s_emptydir)|
|main|kibana.yml|Kibana|1||
|main|fleet.yml|Fleet Server(Agent)|1|[Fleet](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/fleet-managed-elastic-agent), [APM Integration](https://www.elastic.co/docs/solutions/observability/apm/switch-to-elastic-apm-integration) |
|main|fleet.yml|[Agents](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/configuration-examples-fleet)|1+n|[System](https://www.elastic.co/docs/reference/integrations/system), [Kubernetes](https://www.elastic.co/docs/reference/integrations/kubernetes)|
|main|fleet.yml|[RBAC Roles](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)||Kubernetes RBAC for Agents, Fleet|
|ECK-Wide|trial-license.yml|Secret:[License](https://www.elastic.co/docs/deploy-manage/license/manage-your-license-in-eck)||trial-license.yml|[Trial License](https://www.elastic.co/docs/deploy-manage/license/manage-your-license-in-eck#k8s-start-trial)|

## Quickstart Cluster with Ephemeral Storage

This is a basic quickstart ECK cluster that has no persistent storage.  It is meant for drop-in demos or fast exploration with deletion shortly thereafter.

It runs 3 nodes on purpose. ECK upgrades a cluster with fewer than 3 nodes all at once instead of one node at a time ([update strategy](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/update-strategy)). With ephemeral `emptyDir` storage, that would restart every node with empty data at the same time and the cluster could not re-form, so keep at least 3 nodes if you plan to upgrade it.

Fleet and a DaemonSet of Agents are offered as a setup as well.  Each has several integrations you'd want for Kubernetes observability and Kibana is preconfigured to load these Fleet policies. Lastly there is a trial license secret and instructions for deploying `kube-state-metrics` for certain metricsets.

These are just flat yamls that you can deploy at will and as long as the application permits it, leave off anything you don't want. For example:

`kubectl apply -f elasticsearch.yml -f kibana.yml -f fleet.yml -f trial-license.yml`

## Viewing your Data in Kibana

You can use the built-in `elastic` user and the password is stored in a default secret. The command below should pull this value for you. 

`kubectl get secret elasticsearch-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo`

You can then access the running Kibana by running a kubectl port-forward command like below or naturally, working to expose the cluster some other way.

`kubectl port-forward service/kibana-kb-http 5601`


## Adding Kube-State-Metrics

To add the kube-state-metrics server (a Kubernetes official resource) for the `state_*` metricsets, you can visit the repo or deploy the standard example directory resources after cloning the repo locally.

eg:
1. `git clone https://github.com/kubernetes/kube-state-metrics.git`
2. `kubectl apply -f kube-state-metrics/examples/standard/`

## Next Steps?

- Want to see the API specs for each CRD? -> [CRD API Reference](https://www.elastic.co/docs/reference/cloud-on-k8s/api-reference)
- Want to add Elasticsearch cluster settings? -> [Node Configurations](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/node-configuration)
- Want to specify out the cluster resources for a larger cluster?
	+ [Manage Compute Resources](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/manage-compute-resources)
	+ [Virtual Memory Settings](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/virtual-memory)
- Want to expose the cluster via other Services? -> [Kubernetes Services](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/accessing-services)
- Use Your/Custom Certificates -> [Custom Certs](https://www.elastic.co/docs/deploy-manage/security/k8s-https-settings)
- Want to setup persistent storage
	+ [Volume Claim Templates](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/volume-claim-templates)
	+ Bonus Reading: [Storage Recommendations](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/storage-recommendations) 
- Want to setup Elastic cluster snapshots? -> [Create Automated Snapshots](https://www.elastic.co/docs/deploy-manage/tools/snapshot-and-restore/cloud-on-k8s)
- Want to setup a cluster with multiple node types? -> [NodeSets Overview](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/nodes-orchestration)
- Want to setup a more complex cluster topology? -> [Advanced Node Scheduling](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/advanced-elasticsearch-node-scheduling)
- Want to fix/upgrade the license -> [Manage licenses in ECK](https://www.elastic.co/docs/deploy-manage/license/manage-your-license-in-eck)
- Check out our recipe section in the docs -> [Recipes for ECK](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/recipes)

