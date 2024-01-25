load('ext://namespace', 'namespace_create', 'namespace_inject')
namespace_create('monitoring')

print('setup prometheus')
k8s_yaml([
    "k8s/prometheus/clusterRole.yaml",
    "k8s/prometheus/configMap.yaml",
    "k8s/prometheus/deployment.yaml", 
    "k8s/prometheus/service.yaml"
])
k8s_resource('prometheus-deployment', port_forwards=[9090])

print('setup alertmanager')
k8s_yaml([
    "k8s/alertmanager/configMap.yaml",
    "k8s/alertmanager/deployment.yaml", 
    "k8s/alertmanager/service.yaml"
])
k8s_resource('alertmanager', port_forwards=[9093])

print('setup grafana')
k8s_yaml([
    "k8s/grafana/configMap.yaml",
    "k8s/grafana/deployment.yaml", 
    "k8s/grafana/service.yaml"
])
k8s_resource('grafana', port_forwards=[3000])

print('setup node-exporter')
k8s_yaml([
    "k8s/node-exporter/daemonset.yaml",
    "k8s/node-exporter/service.yaml"
])
k8s_resource('node-exporter', port_forwards=[9100])