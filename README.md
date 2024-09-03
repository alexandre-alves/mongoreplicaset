Iremos Criar nosso Mongodb Replica set com dados persistindo em cluster de docker Longhorn

Vamos criar nosso services, Statefulset e nossa String de conexao Mongodb

Mãos a obra:

Vamos criar nosso services
mongodb-services.yaml
kubectl apply -f mongodb-services.yaml

No nosso statefulset você pode alterar o número da réplica pelo seu próprio. 
Eu uso 3 aqui. Após alguns minutos, você deverá ver o statefulset criado.
Vamos criar nosso Statefulset
mongodb-statefulset.yaml

kubectl apply -f mongodb-statefulset.yaml

Você pode vê-lo em seu Rancher ou usando o comando kubectl get. 
Vamos ter certeza de que o Mongodb está sendo executado usando o comando kubectl abaixo.

 kubectl --namespace=default exec -ti mongodb-0 -- mongosh

Então você deve abrir o prompt do mongosh.
Para a segunda e terceira réplica, você pode fazer o mesmo alterando mongodb-0 por mongodb-1 e mongodb-2 Configurar conjunto de réplicas

Para configurar o conjunto de réplicas do MongoDB.
Primeiro precisamos abrir o mongodb-0 (primeiro pod do MongoDb) usando o comando acima. Então usamos o comando rs.initiate() conforme abaixo

rs.initiate(
 {
 _id: "rs0",
 members: [
 { _id: 0, host: "mongodb-0.mongodb-clusterip.default.svc.cluster.local:27017" },
 { _id: 1, host: "mongodb-1.mongodb-clusterip.default.svc.cluster.local:27017" },
 { _id: 2, host: "mongodb-2.mongodb-clusterip.default.svc.cluster.local:27017" }
 ]
 }
)

Se você notar que o padrão é <pod>.<service>.<namespace>.svc.cluster.local:port 
Em seguida, tente adicionar alguns dados ao banco de dados de teste usando a consulta

db.test.insertOne({ testFromMaster: "true" })

Agora saia da réplica master e abra o mongosh na réplica secundária dois dois pods usando o mesmo comando

kubectl --namespace=default exec -ti mongodb-1 -- mongosh

Você notará que o prompt está exibindo o secundário. 
Para começar a ler as alterações de dados da réplica primária, precisamos executar o comando

Lembrando que a forma que ele ira trabalhar vai ser definada da forma que insrir abaixo
veja na documentação do mongo mais opções: Read Preference

No caso estamos preferindo usar a padrão a opção 1

1 - db.getMongo().setReadPref("primary")

2 - db.getMongo().setReadPref("primaryPreferred")

Deve setar ela no dois mongos de replica mongodb-1 e 2

 
Então podemos testar para consultar os dados de teste que criamos antes de usar a consulta

db.test.find({})

 
String de conexao Mongodb
Documentação

Connection Strings

 

mongodb://admin:123456789admin@mongodb-0.mongodb-clusterip.default.svc.cluster.local:27018,mongodb-0.mongodb-clusterip.default.svc.cluster.local:27017,mongodb-0.mongodb-clusterip.default.svc.cluster.local:27019/?authSource=admin&replicaSet=rs0&readPreference=primary


Padrao da conexão mongodb://user:senha:host:port,host:port/?authSource=AuthDB&replicaSet=seu RSinit&readPreference=preferencia de leitura primary ou secundary

OBS: Lembrando que se você não estiver com um DNS para os nomes edite o seu hosts

192.168.0.21 mongodb-0.mongodb-clusterip.default.svc.cluster.local
192.168.0.20 mongodb-1.mongodb-clusterip.default.svc.cluster.local
192.168.0.22 mongodb-2.mongodb-clusterip.default.svc.cluster.local
