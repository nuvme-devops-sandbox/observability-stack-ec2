# Observability Stack

Before anything else, make sure the host has Docker installed.  
If not, install it:

```
sudo apt update
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker ubuntu
newgrp docker
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

Um tamanho adequado para a stack é t4g.medium e t4g.large, 100% compatível com ARM

No grafana você vai precisar ajustar o webhook de alertas e também a url do grafana no notification_template.yaml

1- Passo criar o webhook no slack e colocar a imagem do grafana

2- Alterar o `contact_points.yaml: url: xxx` com a url completa do webhook

3- Alterar o `notification_templates.yaml`, onde está marcado como xxx você colocar a url do cliente apenas

4- Na seção de dashboards, o ajuste é o seguinte: Se o cliente tiver containers, usar os arquivos `ec2_loki_containers_v1.json e ec2_prometheus_host_with_containers_v1.json`, se não usar os outros dois, após a desição deletar os arquivos que não vão ser usados!

5- Uma role assumida pela EC2 de observability com as seguintes policies: 

```AmazonEC2ReadOnlyAccess
AmazonGrafanaCloudWatchAccess

s3-loki
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::loki-client",
                "arn:aws:s3:::loki-client/*"
            ]
        }
    ]
}```

6- loki-s3 bucket permission

```{
    "Version": "2012-10-17",
    "Id": "SSLPolicy",
    "Statement": [
        {
            "Sid": "AllowSSLRequestsOnly",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::loki-client",
                "arn:aws:s3:::loki-client/*"
            ],
            "Condition": {
                "Bool": {
                    "aws:SecureTransport": "false"
                }
            }
        }
    ]
}```

7- Ajustar a configuração do `loki-config.yaml` com o bucket-name e region do cliente

```
bucketnames: loki-client
region: us-east-1
```

8- Caso o cliente use containers na ec2 descomentar o job cadvisor `/prometheus/prometheus.yaml`

9- Adicionar a tag `prometheus=true` na ec2 onde está o node_exporter e o cadvisor, é com isso que o prometheus identifica de quem ele precisa pegar as métricas

10- No .env você vai definir a senha do usuário admin

11- No docker-compose alterar para o tipo de imagem compatível com a arch da máquina, caso use amd altere a imagem para a que está comentada ao lado da atual

```image: grafana/loki:3.5.5-arm64 # image: grafana/loki:3.5.5```

12- docker-compose up -d :pray: