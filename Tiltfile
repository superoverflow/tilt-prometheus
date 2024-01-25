load('ext://namespace', 'namespace_create', 'namespace_inject')
namespace_create('monitoring')

print('setup prometheus')
k8s_yaml([
    "k8s/prometheus/configmap.yaml",
    "k8s/prometheus/deployment.yaml", 
    "k8s/prometheus/service.yaml"
])
k8s_resource('prometheus-deployment', port_forwards=[9090])


print('setup alertmanager')
k8s_yaml([
    "k8s/alertmanager/configmap.yaml",
    "k8s/alertmanager/deployment.yaml", 
    "k8s/alertmanager/service.yaml"
])
k8s_resource('alertmanager', port_forwards=[9093])