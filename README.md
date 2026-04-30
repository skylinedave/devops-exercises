# DevOps Exercise – Kubernetes & Helm

Ho risolto l'esercizio partendo dalla configurazione del cluster Kind, poi ho aggiunto progressivamente i manifest Kubernetes per backend, frontend e gateway. Dopo aver verificato il funzionamento con `kubectl apply`, ho convertito i manifest in un chart Helm parametrizzando immagini, repliche, porte e NodePort.

Le scelte principali sono state:
- usare namespace dedicato `devops`;
- separare frontend, backend e gateway in deployment distinti;
- usare NGINX come reverse proxy nel gateway;
- esporre il gateway via NodePort per semplificare il test locale.

# 1. Configurazione del cluster (kind)

Il cluster è definito nel file:

kind/cluster-config.yaml

Vengono creati:

* un nodo control-plane per gestione
* due nodi worker operativi
* mappature personalizzate delle porte per esporre i servizi a livello locale

Si procede alla creazione del cluster con il seguente comando:

kind create cluster --config kind/cluster-config.yaml

E si verifica lo stato dei nodi appena creati, verificando più dettagli possibili con:

kubectl get nodes -o wide

# 2. Manifest di Kubernetes

I manifest grezzi si trovano in:

k8s/

Includono:

* namespace
* deployment (frontend, backend, gateway)
* service (NodePort)
* configmap per l'instradamento del gateway

Questi file possono essere applicati al cluster e verificati nella seguente maniera:

kubectl apply -f k8s/ (per applicare la repo con le configurazioni)
kubectl get all -n devops (per verificare l'output)

# 3. Architettura dell'applicazione

L'applicazione è suddivisa in tre componenti:

* Un **gateway** che funge da punto di accesso al sistema
* Un **frontend** che gestisce l'interfaccia utente
* Un **backend** che gestisce le richieste API

Tutto il traffico in entrata passa attraverso il gateway (esposto su `localhost:8080`).
Il gateway è configurato per instradare le richieste internamente:

* le richieste a `/` vengono inoltrate al servizio frontend
* le richieste a `/api/` vengono inoltrate al servizio backend

Il frontend e il backend sono inoltre esposti individualmente tramite NodePorts per il testing:

* frontend → `localhost:8081`
* backend → `localhost:8082`

La comunicazione tra i componenti avviene all'interno del cluster utilizzando il service discovery di Kubernetes (`*.svc.cluster.local`).

# 4. Accedere all'app

I servizi sono esposti tramite NodePort, che vengono mappati su localhost tramite la configurazione di Kind.

Ciò significa che puoi accedere a tutto direttamente dal tuo computer:

* gateway → http://localhost:8080
* frontend → http://localhost:8081
* backend → http://localhost:8082

Come test rapido si può utilizzare una curl per chiamare e vedere se viene ricevuta la chiamata.

# 5. Helm chart

L'Helm chart si trova in helm/demo-app e rispecchia i manifesti Kubernetes grezzi.

Invece di hardcodare i valori, elementi come le versioni delle immagini, il numero di repliche e le porte sono configurabili tramite values.yaml.

Per installare la chart ho usato:

helm install demo-app helm/demo-app

Per eventualmente scalare le risorse ora è necessario solo aggiornare i valori tramite il comando di upgrade, piuttosto che modificare daccapo i file.

# 6. Note

- Ho utilizzato i servizi NodePort per semplificare il tutto e allinearmi alle mappature delle porte di Kubernetes.
- Attualmente il gateway usa una configurazione NGINX minimale.
- Non sono stati aggiunti readiness/liveness probe.
- Il chart Helm potrebbe essere migliorato separando meglio i valori di frontend, backend e gateway.
- Non è presente un Ingress controller; NodePort è stato scelto per semplicità locale.
- Non è stata configurata alcuna persistenza, poiché non era richiesta per questo esercizio.
- Il gateway è configurato tramite una ConfigMap anziché integrare la configurazione nell'immagine, il che semplifica l'aggiornamento del routing senza dover ricompilare i container.
