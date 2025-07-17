# CockroachDB - Kubernetes

Para executar o CockroachDB com o Kubernetes é preciso ter o `minikube` e o `kubectl` instalados na sua máquina

## 1. Iniciar o Kubernetes

```
minikube start
```

## 2. Iniciar o CockroachDB

Execute o comando abaixo:
```
kubectl apply -f crds.yaml
```

```
kubectl apply -f operator.yaml
```

```
kubectl config set-context --current --namespace=cockroach-operator-system
```

Para visualizar se o Operator foi iniciado com sucesso execute:
```
kubectl get pods
```
Deve aparecer algo do tipo:
```
NAME                                  READY   STATUS    RESTARTS   AGE
cockroach-operator-6f7b86ffc4-9ppkv   1/1     Running   0          54s

```

Também é possível acompanhar pela dashboard do minikube
```
minikube dashboard
```

## 3. Iniciar Cluster

```
kubectl apply -f deployment.yaml
```

Para visualizar se foi iniciado com sucesso execute:
```
kubectl get pods
```
Pode demorar alguns segundos para todos os pods realmente iniciarem.

## 4. Criação do Usuário

```
kubectl create -f client-secure-operator.yaml
```

```
kubectl exec -it cockroachdb-client-secure \
-- ./cockroach sql \
--certs-dir=/cockroach/cockroach-certs \
--host=cockroachdb-public
```

O nome do usuário é `roach`, substitua `senha` pela senha desejada.
```
CREATE USER roach WITH PASSWORD 'senha';
```
```
GRANT admin TO roach;
```
Para sair
```
\q
```
Essa etapa precisa ser feita apenas uma vez.

## 5. Acessar painel do banco de dados

```
kubectl port-forward service/cockroachdb-public 8080
```
Depois acesse `http://localhost:8080` e realize o login com o usuário e senha criados.


## 6. Acessar pelo DBeaver ou pela aplicação

```
kubectl port-forward service/cockroachdb 26257
```
Informações de acesso

`HOST`: localhost

`USER`: roach (criado na etapa 4)

`PASSWORD`: senha (criada na etapa 4)

`DATABASE_NAME`: system (um novo será criado nesse acesso)

Acesse o DBeaver acesse com o usuário e senha criados na etapa 4.
Nessa etapa acesso com o database padrão `system`.

Após acessar cria um banco de dados com o comando abaixo em um SCRIPT SQL do DBeaver:
```
create database hapvida;
```

Agora você pode alterar a conexão para usar o banco de dados criado acima.

Você pode criar tabelas também:

```
create table hapvida.leads (
	id uuid primary key,
	nome text,
	email text,
	telefone text,
    created_at timestamp
);

create table hapvida.budgets (
	id uuid primary key,
	lead_id uuid,
	utm_info jsonb,
	jornada_timestamp jsonb,
	respostas jsonb,
	ia_rede jsonb,
	ia_planos jsonb,
	ia_planos_timestamp timestamp,
	ia_planos_success boolean,
	ia_script jsonb,
	created_at timestamp,
	tipo_contratacao text,
	devaice_info jsonb,
	finalizado boolean,
	ultima_etapa text,
	pref_contato text,
	beneficios_interesse jsonb,
	planos_interesse jsonb
);
```

Agora é possível acessar pela aplicação e manipular os dados nas tabelas (use o `DATABASE_NAME` em que as tabelas foram criadas)