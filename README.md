# DevOps Exercise – Kubernetes & Helm

Questo progetto configura un ambiente Kubernetes locale utilizzando kind, deploya una semplice applicazione tramite manifest Kubernetes e, infine, converte la configurazione in un Helm chart.

L'applicazione è composta da:

* un frontend (nginx)
* un backend (HTTP)
* un gateway (nginx che funge da proxy inverso)

# 1. Configurazione del cluster (kind)

Il cluster è definito nel file:

kind/cluster-config.yaml

Vengono creati:

* 1 nodo control-plane per gestione
* 2 nodi worker operativi
* mappature personalizzate delle porte per esporre i servizi a livello locale

Si procede alla creazione del cluster

# 2. Manifest di Kubernetes

I manifest grezzi si trovano in:

k8s/

Includono:

* namespace
* deployment (frontend, backend, gateway)
* service (NodePort)
* configmap per l'instradamento del gateway

Questi file possono essere applicati al cluster e verificati

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

# 5. Helm chart

L'Helm chart si trova in helm/demo-app e rispecchia i manifesti Kubernetes grezzi.

Invece di hardcodare i valori, elementi come le versioni delle immagini, il numero di repliche e le porte sono configurabili tramite values.yaml.

# 6. Note

Ho utilizzato i servizi NodePort per semplificare il tutto e allinearmi alle mappature delle porte di Kubernetes.

Non è stata configurata alcuna persistenza, poiché non era richiesta per questo esercizio.

Il gateway è configurato tramite una ConfigMap anziché integrare la configurazione nell'immagine, il che semplifica l'aggiornamento del routing senza dover ricompilare i container.

Ho inoltre aggiunto dei selettori di nodi in modo che il frontend e il backend possano essere distribuiti su nodi di lavoro diversi.
