load('ext://namespace', 'namespace_create', 'namespace_inject')
namespace_create('monitoring')

print('setup prometheus')
k8s_yaml([
    "k8s/prometheus/configmap.yaml",
    "k8s/prometheus/deployment.yaml", 
    "k8s/prometheus/service.yaml"]
)
k8s_resource('prometheus-deployment', port_forwards=[9090])