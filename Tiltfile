load('ext://namespace', 'namespace_create', 'namespace_inject')
load('ext://deployment', 'deployment_create')
load('ext://configmap', 'configmap_from_dict')
load('ext://uibutton', 'cmd_button')

allow_k8s_contexts(['docker-desktop', 'colima'])

namespace_create('monitoring')
namespace_create('database')

def setup_prometheus():
    k8s_yaml([
        "k8s/prometheus/clusterRole.yaml",
        "k8s/prometheus/configMap.yaml",
        "k8s/prometheus/deployment.yaml", 
        "k8s/prometheus/service.yaml"
    ])
    k8s_resource('prometheus', port_forwards=[9090])

def setup_alertmanager():
    k8s_yaml([
        "k8s/alertmanager/configMap.yaml",
        "k8s/alertmanager/deployment.yaml", 
        "k8s/alertmanager/service.yaml"
    ])
    k8s_resource('alertmanager', port_forwards=[9093])

def setup_grafana():
    k8s_yaml([
        "k8s/grafana/configMap.yaml",
        "k8s/grafana/deployment.yaml", 
        "k8s/grafana/service.yaml"
    ])
    k8s_resource('grafana', port_forwards=[3000])

def setup_node_exporter():
    k8s_yaml([
        "k8s/node-exporter/daemonset.yaml",
        "k8s/node-exporter/service.yaml"
    ])
    k8s_resource('node-exporter', port_forwards=[9100])


def setup_postgres(instance_number=0):
    deployment_name='postgres-' + str(instance_number)
    namespace='database'
    postgres_user='postgres'
    postgres_password='postgres'
    postgres_config_name='postgres-env-' + str(instance_number)
    expose_port=5432 + instance_number
    
    k8s_yaml(
        configmap_from_dict(
            name=postgres_config_name, 
            namespace=namespace, 
            inputs={
                'POSTGRES_PASSWORD': postgres_password
            }
        )
    )

    deployment_create(
        name=deployment_name,
        image='postgres:14.0-alpine',
        namespace=namespace,
        ports=['5432:5432'],
        envFrom=[
                {   
                    'configMapRef': {
                        'name': postgres_config_name
                    }
                }
            ]
        )
    k8s_resource(deployment_name, port_forwards=[expose_port])

    pod_exec_script = '''
    set -eu
    POD_NAME="$(tilt get kubernetesdiscovery {service} -ojsonpath='{{.status.pods[0].name}}')"
    kubectl exec -n database $POD_NAME -- psql -h localhost -U {user} -c 'SELECT pg_sleep(20)'
    '''.format(service=deployment_name, user=postgres_user)
    cmd_button('postgres:stress:' + str(instance_number),
        resource=deployment_name,
        argv=['sh', '-c', pod_exec_script],
        icon_name='fitness_center',
        text='stress',
    )



def setup_postgres_exporter(instance_number=0):
    print('setup postgres exporter')
    namespace = 'monitoring'
    deployment_name = 'postgres-exporter-' + str(instance_number)
    database_name = 'postgres-' + str(instance_number)
    postgres_user = 'postgres'
    postgres_password = 'postgres'
    postgres_exporter_config_name = 'postgres-exporter-env' + str(instance_number)
    postgres_connection_str = (
        'postgresql://' + postgres_user + ':' + postgres_password +
        '@' + database_name + '.database.svc:5432/postgres?sslmode=disable'
    )
    port = 9187 + instance_number
    k8s_yaml(
        configmap_from_dict(
            name=postgres_exporter_config_name, 
            namespace='monitoring', 
            inputs={
                'DATA_SOURCE_NAME': postgres_connection_str
            }
        )
    )
    deployment_create(
        name=deployment_name,
        image='quay.io/prometheuscommunity/postgres-exporter',
        namespace='monitoring',
        envFrom=[
            {   
                'configMapRef': {
                    'name': postgres_exporter_config_name
                }
            }
        ],
        ports=['9187:9187']
    )
    k8s_resource(deployment_name, port_forwards=[port])




def main():
    setup_prometheus()
    setup_alertmanager()
    setup_grafana()

    setup_node_exporter()
    for i in range(0, 5):
        setup_postgres(instance_number=i)
        setup_postgres_exporter(instance_number=i)

main()

