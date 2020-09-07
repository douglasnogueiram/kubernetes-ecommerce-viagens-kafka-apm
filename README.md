# Kubernetes-ecommerce-viagens-Kafka
 Arquivos YAML para configuração K8s e Istio. Nesta versão, a integração via RabbitMQ é substituida pelo Apache Kafka. Também foi incluído o monitoramento via Elastic ALM.

![alt text](https://github.com/douglasnogueiram/kubernetes-ecommerce-viagens-kafka-apm/blob/master/ecommerce-kafka/Ecommerce_Kafka_APM.png)

Os microsserviços de exemplo estão nos repositórios:
- https://github.com/douglasnogueiram/ms-communication-bank-kafka
- https://github.com/douglasnogueiram/ms-communication-buyfeedback-kafka
- https://github.com/douglasnogueiram/ms-communication-buyprocess-kafka
- https://github.com/douglasnogueiram/ms-communication-buytrip-kafka

Para fazer os testes, seguir os passos:

1. Gerar o build dos componentes, na pasta ms-communication-XXX (repetir esse passo para cada MS existente). O build está configurado para já gerar uma imagem Docker e subir par o Docker Hub, como mostrado no artigo https://medium.com/@fernandoevangelista_28291/criando-e-enviando-imagem-docker-com-java-e-maven-4fa3c70dba0f
```
$ mvn clean package dockerfile:push
```

2. Confirmar se imagens subiram para Docker Hub


3. Subir minikube e Istio
```
$ minikube start --memory=4096 --cpus=3
$ minikube dashboard

$ export PATH=$pwd/bin:$PATH

$ istioctl install --set profile=demo
$ kubectl label namespace default istio-injection=enabled
$ kubectl apply -f istio-1.7.0/samples/addons
$ istioctl dashboard kiali
```
Kiali: user "admin"    senha "admin"


4. Subir serviços básicos
```
$ kubectl create -f mysql.yml

$ kubectl create -f rabbitmq.yml
$ minikube service rs-rmq-mgt

$ kubectl create -f redis.yml
```

5. Acompanhar os logs de execução via Dashboard Kubernetes ou via terminal
```
$ kubectl logs -t <nome do pod>
```
6. Subir o cluster do Apache Kafka e criar os tópicos abaixo. As instruções para a criação do cluster local estão na página oficial (https://kafka.apache.org/quickstart).
```
$ pagamento-aguardando
$ pagamento-finalizado

```

7. Subir os componentes da suíte Elastic, no diretório "Elastic":
```
$ kubectl apply -f all-in-one.yaml
$ kubectl apply -f elasticsearch.yaml
$ kubectl apply -f kibana.yaml
$ kubectl apply -f apmserver.yaml

```
8. Gerar o secret para acessar o Kibana e APM Server (client). As orientações gerais estão na página oficial (https://www.elastic.co/guide/en/cloud-on-k8s/1.0/index.html), mas os principais passos estão descritos a seguir:
```
$ kubectl get service quickstart-es-http
$ PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode)
$ kubectl port-forward service/quickstart-es-http 9200
$ kubectl get service quickstart-kb-http
$ kubectl port-forward service/quickstart-kb-http 5601
$ kubectl create secret generic apm-secret-settings --from-literal=ES_PASSWORD=asecretpassword
$ kubectl create secret generic es-ca --from-file=tls.crt=elasticsearch-ca.crt
$ kubectl get secret/apm-server-quickstart-apm-token -o go-template='{{index .data "secret-token" | base64decode}}'

```
9. O secret do APM Server deve ser alterado em cada arquivo YAML, no atributo ```ELASTIC_APM_SECRET_TOKEN```.

10. Subir MS Banco
```
$ kubectl create -f bank.yml
$ kubectl create -f bank-v2.yml
$ minikube service service-bank --url
```

11. Subir MS buyfeedback
```
$ kubectl create -f buyfeedback.yml
$ minikube service service-buyfeedback --url
```

12. Subir MS buyprocess
```
$ kubectl create -f buyprocess.yml
$ minikube service service-buyprocess --url
```

13. Subir MS buytrip
```
$ kubectl create -f buytrip.yml
$ kubectl create -f buytrip-v2.yml
$ minikube service service-buytrip --url
```

14. Subir o gateway e os virtualservices
```
$ kubectl create -f gateway.yml

$ kubectl create -f destinationrule-bank.yml
$ kubectl create -f destinationrule-buytrip.yml

$ kubectl create -f virtualservices-bank.yml
$ kubectl create -f virtualservices-buytrip.yml
```
