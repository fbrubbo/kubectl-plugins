# kubectl-plugins

To start using these plugins, first execute the following:

```bash
git clone https://github.com/fbrubbo/kubectl-plugins.git
export PATH=$PATH:$(pwd)/kubectl-plugins
kubectl snapshot -h
```

## kubectl snapshot

**Disclaimer: this pluting uses *kubectl top* to get cpu and memory usage. That means, it gets current resource usage. In other workds, this plugin does not consider historical data.**

To take a snapshot of pods, hpas, deployments without hpas and nodes

```bash
kubectl snapshot
```

You can also filter by namespace, deployment or pod (or a mix of them)

```bash
kubectl snapshot -n my-ns
kubectl snapshot -d my-deployment
kubectl snapshot -p my-pod
```

To check all option type

```bash
kubectl snapshot -h
```

The default behaviour of this plugin is to print the output in the stdio, if you would like to generate csv files to import it to a spreadsheet use

```bash
kubectl snapshot -csv-output <snapshot-name>
```

The above command will generate 4 files

- **kubectl-snapshot-\<DATE_TIME\>-pods.csv** : all pods data and its respective resource usage
- **kubectl-snapshot-\<DATE_TIME\>-hpas.csv** : all hpas data and all its pods respective resource usage
- **kubectl-snapshot-\<DATE_TIME\>-nohpas.csv** : all deploymentes without hpa and all its pods respective resource usage
- **kubectl-snapshot-\<DATE_TIME\>-nodes.csv** : all nodes data and all its respective resource usage

### Sugestions on how to interpret the data

1. Start by taking a snapshot with **-csv-output** parameter
2. Import all files in a spreadsheet (use different sheets for each file)
3. In the **hpa** sheet, sort by **Usage CPU (%)** in the acendent order
   - You will see at the top hpas which, probably, are miss configured
      - Or your app is not being used at the time you run this command
      - Or min replicas too big
      - Or target cpu too low
      - Or the request resources in de pods are too high
   - Another good aproach is to filter only hpa with low **Usage CPU (%)**, eg < 55%, and then order **Requests CPU (m)** in decendent order.
      - You will see at the top, the top offenders.
4. In the **nohpa** sheet, sort by **Usage CPU (%)** in the acendent order
   - You will see at the top deployments which are requesting a lot of resource and not using it. Consider:
       1. reducing the resources requests
       2. and creating an HPA for each of them
   - The same way as HPA, you can filter low usage and then look at top offenders.
5. In the **nohpa** sheet, sort by **Usage CPU (%)** in the decendent order
   - You will see at the top deployments which are using a lot of resources. Consider:
       1. Check if resource requets and limits are set. When it is not set, the plugin set 100%. In this case, consider seting the appropriate values.
       2. Revisit the app to see why it is using too much resources
       3. If your app is already doing its best, increase the resources requests
6. In the **node** sheet
   - Look at the **Allocated** vs  **Actual** info to understand if you have nodes underutilized
     - If so, you may have too high min value in node pools, over used memory but not cpu, etc
   - Using columns **Request CPU (m)** and **Top CPU (m)**, you can do a simple math to have an approximation of how much you are spending above what you need. Note that in stdoi output, this math is already done for you
   - Similarly, you can use **Allocated CPU (m)** and **Request CPU (m)** to understand how much your capacity is bigger than you requested.
   - Use the **-debug** parameber to understand which pods are in which node (stdio only)
7. Use the **pods** sheet for fine tunning

Note that this is only a suggetion, you can do many other similar analysis to improve the usage of your cluster and, consequently, reduce your costs.

## kubectl snapshot hpas

Shortcut for

```bash
kubectl snapshot -print hpas
```

## kubectl snapshot nodes

Shortcut for

```bash
kubectl snapshot -print nodes
```

## kubectl snapshot pods

Shortcut for

```bash
kubectl snapshot -print pods
```

## Other plugins

Do not use other plugins, they are crap.
