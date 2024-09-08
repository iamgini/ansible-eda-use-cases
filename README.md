# Event-Driven-Kubernets-with-Ansible

Pre-req

```shell
$ pip install kubernetes
```

Install ansible-rulebook

```shell
dnf --assumeyes install java-17-openjdk python3-pip
export JAVA_HOME=/usr/lib/jvm/jre-17-openjdk
pip3 install ansible ansible-rulebook ansible-runner
```

## Dev Kubernetes cluster using minikube

```shell
$ minikube start \
    --driver=virtualbox \
    --nodes 1 \
    --cni calico \
    --cpus=2 \
    --memory=2g \
    --kubernetes-version=v1.31.0 \
    --container-runtime=containerd \
    --profile k8s-1-31
```


```yaml
- name: Automatic Remediation of a web server
  hosts: all
  sources:
    - name: listen for alerts
      ansible.eda.alertmanager:
        host: 0.0.0.0
        port: 8000
  rules:
    - name: restart web server
      condition: event.alert.labels.job == "fastapi" and event.alert.status == "firing"
      action:
        run_playbook:
          name: ansible.eda.start_app
```


```shell
$ ansible-galaxy collection install sabre1041.eda
```

```shell
ansible-rulebook -i host --rulebook k8s-rulebook.yaml --verbose
```

Create a configmap to test

```shell
$ kubectl create configmap -n default eda-example --from-literal=message="Kubernetes Meets Event-Driven Ansible"
configmap/eda-example created
```

## Troubleshooting


Missing libs

```shell
$  ansible-rulebook --version
...
_libpq
    raise ImportError(
ImportError: no pq wrapper available.
Attempts made:
- couldn't import psycopg 'c' implementation: No module named 'psycopg_c'
- couldn't import psycopg 'binary' implementation: No module named 'psycopg_binary'
- couldn't import psycopg 'python' implementation: libpq library not found
```

```shell
$ sudo dnf install postgresql-devel
```