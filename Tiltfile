load('ext://namespace', 'namespace_create', 'namespace_inject')
load('ext://deployment', 'deployment_create')
load('ext://configmap', 'configmap_from_dict')
allow_k8s_contexts(['docker-desktop', 'colima'])

namespace_create('monitoring')
namespace_create('database')

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

print('setup progres')
k8s_yaml(
    configmap_from_dict("postgres-env", 
        namespace='database', 
        inputs={
            'POSTGRES_PASSWORD': 'postgres'
        }))
deployment_create(
  'postgres',
  image='postgres:14.0-alpine',
  namespace='database',
  ports=['5432:5432'],
  envFrom=[
    {   
        'configMapRef': {
            'name': 'postgres-env'
        }
    }]
)
k8s_resource('postgres', port_forwards=[5432])

print('setup postgres exporter')
k8s_yaml(
    configmap_from_dict("postgres-exporter-env", 
        namespace='monitoring', 
        inputs={
            'DATA_SOURCE_NAME': 'postgresql://postgres:postgres@postgres.database.svc:5432/postgres?sslmode=disable'
        }))
deployment_create(
  'postgres-exporter',
  image='quay.io/prometheuscommunity/postgres-exporter',
  namespace='monitoring',
  envFrom=[
    {   
        'configMapRef': {
            'name': 'postgres-exporter-env'
        }
    }],
    ports=['9187:9187']
)
k8s_resource('postgres-exporter', port_forwards=[9187])