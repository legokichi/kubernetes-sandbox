
## linkerd2(Conduit)
* 改札サイト一覧 - https://github.com/linkerd/website
* https://qiita.com/mamomamo/items/92085e0e508e18bc8532
* https://qiita.com/hirofumimatsu/items/5f8193c2c4a10c509466
* https://linkerd.io/2/overview/
* https://blog.buoyant.io/2017/12/05/introducing-conduit/#_ga=2.53615917.42354775.1535893854-1043067524.1531462563

aws lambda でラムダの名前を指定してラムダ起動したりするけど、その名前を指定したときに対象のラムダがクラウドの中のどのノードにいるのかを瞬時に調べて送受信を面倒みてくれるレイヤ。

* Istio、Conduit、Linkerdなどのサービスメッシュは、Kubernetes Services API とどう違うのですか？ - https://www.quora.com/How-are-service-meshes-such-as-Istio-Conduit-and-Linkerd-different-from-the-Kubernetes-Services-API
* サーキットブレーカー、ロギング、ロードバランス、レートリミット、その他・・・
* kubernetes service は単に名前解決しかしてくれない

```bash
minikube start
minikube dashboard
kubectl version --short
curl https://run.linkerd.io/install | sh
export PATH=$PATH:$HOME/.linkerd2/bin
linkerd version
linkerd install | kubectl apply -f -
kubectl -n linkerd get pods
linkerd dashboard
linkerd dashboard --show linkerd
linkerd dashboard --show grafana
curl https://run.linkerd.io/emojivoto.yml \
  | linkerd inject - \
  | kubectl apply -f -
minikube -n emojivoto service web-svc --url
linkerd -n emojivoto stat deploy
linkerd -n emojivoto tap deploy
minikube stop
```

* dov - https://linkerd.io/2/inject-reference/

difference between `curl https://run.linkerd.io/emojivoto.yml` to `curl https://run.linkerd.io/emojivoto.yml | linkerd inject -`


`diff -w -y a b`

```diff
---                                                             ---
apiVersion: v1                                                  apiVersion: v1
kind: Namespace                                                 kind: Namespace
metadata:                                                       metadata:
  name: emojivoto                                                 name: emojivoto
---                                                             ---
apiVersion: apps/v1beta1                                        apiVersion: apps/v1beta1
kind: Deployment                                                kind: Deployment
metadata:                                                       metadata:
  creationTimestamp: null                                         creationTimestamp: null
  name: emoji                                                     name: emoji
  namespace: emojivoto                                            namespace: emojivoto
spec:                                                           spec:
  replicas: 1                                                     replicas: 1
  selector:                                                       selector:
    matchLabels:                                                    matchLabels:
      app: emoji-svc                                                  app: emoji-svc
  strategy: {}                                                    strategy: {}
  template:                                                       template:
    metadata:                                                       metadata:
                                                              >       annotations:
                                                              >         linkerd.io/created-by: linkerd/cli v18.8.4
                                                              >         linkerd.io/proxy-version: v18.8.4
      creationTimestamp: null                                         creationTimestamp: null
      labels:                                                         labels:
        app: emoji-svc                                                  app: emoji-svc
                                                              >         linkerd.io/control-plane-ns: linkerd
                                                              >         linkerd.io/proxy-deployment: emoji
    spec:                                                           spec:
      containers:                                                     containers:
      - env:                                                          - env:
        - name: GRPC_PORT                                               - name: GRPC_PORT
          value: "8080"                                                   value: "8080"
        image: buoyantio/emojivoto-emoji-svc:v5                         image: buoyantio/emojivoto-emoji-svc:v5
        name: emoji-svc                                                 name: emoji-svc
        ports:                                                          ports:
        - containerPort: 8080                                           - containerPort: 8080
          name: grpc                                                      name: grpc
        resources: {}                                                   resources: {}
                                                              >       - env:
                                                              >         - name: LINKERD2_PROXY_LOG
                                                              >           value: warn,linkerd2_proxy=info
                                                              >         - name: LINKERD2_PROXY_BIND_TIMEOUT
                                                              >           value: 10s
                                                              >         - name: LINKERD2_PROXY_CONTROL_URL
                                                              >           value: tcp://proxy-api.linkerd.svc.cluster.local:80
                                                              >         - name: LINKERD2_PROXY_CONTROL_LISTENER
                                                              >           value: tcp://0.0.0.0:4190
                                                              >         - name: LINKERD2_PROXY_METRICS_LISTENER
                                                              >           value: tcp://0.0.0.0:4191
                                                              >         - name: LINKERD2_PROXY_PRIVATE_LISTENER
                                                              >           value: tcp://127.0.0.1:4140
                                                              >         - name: LINKERD2_PROXY_PUBLIC_LISTENER
                                                              >           value: tcp://0.0.0.0:4143
                                                              >         - name: LINKERD2_PROXY_POD_NAMESPACE
                                                              >           valueFrom:
                                                              >             fieldRef:
                                                              >               fieldPath: metadata.namespace
                                                              >         image: gcr.io/linkerd-io/proxy:v18.8.4
                                                              >         imagePullPolicy: IfNotPresent
                                                              >         livenessProbe:
                                                              >           httpGet:
                                                              >             path: /metrics
                                                              >             port: 4191
                                                              >           initialDelaySeconds: 10
                                                              >         name: linkerd-proxy
                                                              >         ports:
                                                              >         - containerPort: 4143
                                                              >           name: linkerd-proxy
                                                              >         - containerPort: 4191
                                                              >           name: linkerd-metrics
                                                              >         readinessProbe:
                                                              >           httpGet:
                                                              >             path: /metrics
                                                              >             port: 4191
                                                              >           initialDelaySeconds: 10
                                                              >         resources: {}
                                                              >         securityContext:
                                                              >           runAsUser: 2102
                                                              >         terminationMessagePolicy: FallbackToLogsOnError
                                                              >       initContainers:
                                                              >       - args:
                                                              >         - --incoming-proxy-port
                                                              >         - "4143"
                                                              >         - --outgoing-proxy-port
                                                              >         - "4140"
                                                              >         - --proxy-uid
                                                              >         - "2102"
                                                              >         - --inbound-ports-to-ignore
                                                              >         - 4190,4191
                                                              >         image: gcr.io/linkerd-io/proxy-init:v18.8.4
                                                              >         imagePullPolicy: IfNotPresent
                                                              >         name: linkerd-init
                                                              >         resources: {}
                                                              >         securityContext:
                                                              >           capabilities:
                                                              >             add:
                                                              >             - NET_ADMIN
                                                              >           privileged: false
                                                              >         terminationMessagePolicy: FallbackToLogsOnError
status: {}                                                      status: {}
---                                                             ---
apiVersion: v1                                                  apiVersion: v1
kind: Service                                                   kind: Service
metadata:                                                       metadata:
  name: emoji-svc                                                 name: emoji-svc
  namespace: emojivoto                                            namespace: emojivoto
spec:                                                           spec:
  selector:                                                       selector:
    app: emoji-svc                                                  app: emoji-svc
  clusterIP: None                                                 clusterIP: None
  ports:                                                          ports:
  - name: grpc                                                    - name: grpc
    port: 8080                                                      port: 8080
    targetPort: 8080                                                targetPort: 8080
---                                                             ---
apiVersion: apps/v1beta1                                        apiVersion: apps/v1beta1
kind: Deployment                                                kind: Deployment
metadata:                                                       metadata:
  creationTimestamp: null                                         creationTimestamp: null
  name: voting                                                    name: voting
  namespace: emojivoto                                            namespace: emojivoto
spec:                                                           spec:
  replicas: 1                                                     replicas: 1
  selector:                                                       selector:
    matchLabels:                                                    matchLabels:
      app: voting-svc                                                 app: voting-svc
  strategy: {}                                                    strategy: {}
  template:                                                       template:
    metadata:                                                       metadata:
                                                              >       annotations:
                                                              >         linkerd.io/created-by: linkerd/cli v18.8.4
                                                              >         linkerd.io/proxy-version: v18.8.4
      creationTimestamp: null                                         creationTimestamp: null
      labels:                                                         labels:
        app: voting-svc                                                 app: voting-svc
                                                              >         linkerd.io/control-plane-ns: linkerd
                                                              >         linkerd.io/proxy-deployment: voting
    spec:                                                           spec:
      containers:                                                     containers:
      - env:                                                          - env:
        - name: GRPC_PORT                                               - name: GRPC_PORT
          value: "8080"                                                   value: "8080"
        image: buoyantio/emojivoto-voting-svc:v5                        image: buoyantio/emojivoto-voting-svc:v5
        name: voting-svc                                                name: voting-svc
        ports:                                                          ports:
        - containerPort: 8080                                           - containerPort: 8080
          name: grpc                                                      name: grpc
        resources: {}                                                   resources: {}
                                                              >       - env:
                                                              >         - name: LINKERD2_PROXY_LOG
                                                              >           value: warn,linkerd2_proxy=info
                                                              >         - name: LINKERD2_PROXY_BIND_TIMEOUT
                                                              >           value: 10s
                                                              >         - name: LINKERD2_PROXY_CONTROL_URL
                                                              >           value: tcp://proxy-api.linkerd.svc.cluster.local:80
                                                              >         - name: LINKERD2_PROXY_CONTROL_LISTENER
                                                              >           value: tcp://0.0.0.0:4190
                                                              >         - name: LINKERD2_PROXY_METRICS_LISTENER
                                                              >           value: tcp://0.0.0.0:4191
                                                              >         - name: LINKERD2_PROXY_PRIVATE_LISTENER
                                                              >           value: tcp://127.0.0.1:4140
                                                              >         - name: LINKERD2_PROXY_PUBLIC_LISTENER
                                                              >           value: tcp://0.0.0.0:4143
                                                              >         - name: LINKERD2_PROXY_POD_NAMESPACE
                                                              >           valueFrom:
                                                              >             fieldRef:
                                                              >               fieldPath: metadata.namespace
                                                              >         image: gcr.io/linkerd-io/proxy:v18.8.4
                                                              >         imagePullPolicy: IfNotPresent
                                                              >         livenessProbe:
                                                              >           httpGet:
                                                              >             path: /metrics
                                                              >             port: 4191
                                                              >           initialDelaySeconds: 10
                                                              >         name: linkerd-proxy
                                                              >         ports:
                                                              >         - containerPort: 4143
                                                              >           name: linkerd-proxy
                                                              >         - containerPort: 4191
                                                              >           name: linkerd-metrics
                                                              >         readinessProbe:
                                                              >           httpGet:
                                                              >             path: /metrics
                                                              >             port: 4191
                                                              >           initialDelaySeconds: 10
                                                              >         resources: {}
                                                              >         securityContext:
                                                              >           runAsUser: 2102
                                                              >         terminationMessagePolicy: FallbackToLogsOnError
                                                              >       initContainers:
                                                              >       - args:
                                                              >         - --incoming-proxy-port
                                                              >         - "4143"
                                                              >         - --outgoing-proxy-port
                                                              >         - "4140"
                                                              >         - --proxy-uid
                                                              >         - "2102"
                                                              >         - --inbound-ports-to-ignore
                                                              >         - 4190,4191
                                                              >         image: gcr.io/linkerd-io/proxy-init:v18.8.4
                                                              >         imagePullPolicy: IfNotPresent
                                                              >         name: linkerd-init
                                                              >         resources: {}
                                                              >         securityContext:
                                                              >           capabilities:
                                                              >             add:
                                                              >             - NET_ADMIN
                                                              >           privileged: false
                                                              >         terminationMessagePolicy: FallbackToLogsOnError
status: {}                                                      status: {}
---                                                             ---
apiVersion: v1                                                  apiVersion: v1
kind: Service                                                   kind: Service
metadata:                                                       metadata:
  name: voting-svc                                                name: voting-svc
  namespace: emojivoto                                            namespace: emojivoto
spec:                                                           spec:
  selector:                                                       selector:
    app: voting-svc                                                 app: voting-svc
  clusterIP: None                                                 clusterIP: None
  ports:                                                          ports:
  - name: grpc                                                    - name: grpc
    port: 8080                                                      port: 8080
    targetPort: 8080                                                targetPort: 8080
---                                                             ---
apiVersion: apps/v1beta1                                        apiVersion: apps/v1beta1
kind: Deployment                                                kind: Deployment
metadata:                                                       metadata:
  creationTimestamp: null                                         creationTimestamp: null
  name: web                                                       name: web
  namespace: emojivoto                                            namespace: emojivoto
spec:                                                           spec:
  replicas: 1                                                     replicas: 1
  selector:                                                       selector:
    matchLabels:                                                    matchLabels:
      app: web-svc                                                    app: web-svc
  strategy: {}                                                    strategy: {}
  template:                                                       template:
    metadata:                                                       metadata:
                                                              >       annotations:
                                                              >         linkerd.io/created-by: linkerd/cli v18.8.4
                                                              >         linkerd.io/proxy-version: v18.8.4
      creationTimestamp: null                                         creationTimestamp: null
      labels:                                                         labels:
        app: web-svc                                                    app: web-svc
                                                              >         linkerd.io/control-plane-ns: linkerd
                                                              >         linkerd.io/proxy-deployment: web
    spec:                                                           spec:
      containers:                                                     containers:
      - env:                                                          - env:
        - name: WEB_PORT                                                - name: WEB_PORT
          value: "80"                                                     value: "80"
        - name: EMOJISVC_HOST                                           - name: EMOJISVC_HOST
          value: emoji-svc.emojivoto:8080                                 value: emoji-svc.emojivoto:8080
        - name: VOTINGSVC_HOST                                          - name: VOTINGSVC_HOST
          value: voting-svc.emojivoto:8080                                value: voting-svc.emojivoto:8080
        - name: INDEX_BUNDLE                                            - name: INDEX_BUNDLE
          value: dist/index_bundle.js                                     value: dist/index_bundle.js
        image: buoyantio/emojivoto-web:v5                               image: buoyantio/emojivoto-web:v5
        name: web-svc                                                   name: web-svc
        ports:                                                          ports:
        - containerPort: 80                                             - containerPort: 80
          name: http                                                      name: http
        resources: {}                                                   resources: {}
                                                              >       - env:
                                                              >         - name: LINKERD2_PROXY_LOG
                                                              >           value: warn,linkerd2_proxy=info
                                                              >         - name: LINKERD2_PROXY_BIND_TIMEOUT
                                                              >           value: 10s
                                                              >         - name: LINKERD2_PROXY_CONTROL_URL
                                                              >           value: tcp://proxy-api.linkerd.svc.cluster.local:80
                                                              >         - name: LINKERD2_PROXY_CONTROL_LISTENER
                                                              >           value: tcp://0.0.0.0:4190
                                                              >         - name: LINKERD2_PROXY_METRICS_LISTENER
                                                              >           value: tcp://0.0.0.0:4191
                                                              >         - name: LINKERD2_PROXY_PRIVATE_LISTENER
                                                              >           value: tcp://127.0.0.1:4140
                                                              >         - name: LINKERD2_PROXY_PUBLIC_LISTENER
                                                              >           value: tcp://0.0.0.0:4143
                                                              >         - name: LINKERD2_PROXY_POD_NAMESPACE
                                                              >           valueFrom:
                                                              >             fieldRef:
                                                              >               fieldPath: metadata.namespace
                                                              >         image: gcr.io/linkerd-io/proxy:v18.8.4
                                                              >         imagePullPolicy: IfNotPresent
                                                              >         livenessProbe:
                                                              >           httpGet:
                                                              >             path: /metrics
                                                              >             port: 4191
                                                              >           initialDelaySeconds: 10
                                                              >         name: linkerd-proxy
                                                              >         ports:
                                                              >         - containerPort: 4143
                                                              >           name: linkerd-proxy
                                                              >         - containerPort: 4191
                                                              >           name: linkerd-metrics
                                                              >         readinessProbe:
                                                              >           httpGet:
                                                              >             path: /metrics
                                                              >             port: 4191
                                                              >           initialDelaySeconds: 10
                                                              >         resources: {}
                                                              >         securityContext:
                                                              >           runAsUser: 2102
                                                              >         terminationMessagePolicy: FallbackToLogsOnError
                                                              >       initContainers:
                                                              >       - args:
                                                              >         - --incoming-proxy-port
                                                              >         - "4143"
                                                              >         - --outgoing-proxy-port
                                                              >         - "4140"
                                                              >         - --proxy-uid
                                                              >         - "2102"
                                                              >         - --inbound-ports-to-ignore
                                                              >         - 4190,4191
                                                              >         image: gcr.io/linkerd-io/proxy-init:v18.8.4
                                                              >         imagePullPolicy: IfNotPresent
                                                              >         name: linkerd-init
                                                              >         resources: {}
                                                              >         securityContext:
                                                              >           capabilities:
                                                              >             add:
                                                              >             - NET_ADMIN
                                                              >           privileged: false
                                                              >         terminationMessagePolicy: FallbackToLogsOnError
status: {}                                                      status: {}
---                                                             ---
apiVersion: v1                                                  apiVersion: v1
kind: Service                                                   kind: Service
metadata:                                                       metadata:
  name: web-svc                                                   name: web-svc
  namespace: emojivoto                                            namespace: emojivoto
spec:                                                           spec:
  type: LoadBalancer                                              type: LoadBalancer
  selector:                                                       selector:
    app: web-svc                                                    app: web-svc
  ports:                                                          ports:
  - name: http                                                    - name: http
    port: 80                                                        port: 80
    targetPort: 80                                                  targetPort: 80
---                                                             ---
apiVersion: apps/v1beta1                                        apiVersion: apps/v1beta1
kind: Deployment                                                kind: Deployment
metadata:                                                       metadata:
  creationTimestamp: null                                         creationTimestamp: null
  name: vote-bot                                                  name: vote-bot
  namespace: emojivoto                                            namespace: emojivoto
spec:                                                           spec:
  replicas: 1                                                     replicas: 1
  selector:                                                       selector:
    matchLabels:                                                    matchLabels:
      app: vote-bot                                                   app: vote-bot
  strategy: {}                                                    strategy: {}
  template:                                                       template:
    metadata:                                                       metadata:
                                                              >       annotations:
                                                              >         linkerd.io/created-by: linkerd/cli v18.8.4
                                                              >         linkerd.io/proxy-version: v18.8.4
      creationTimestamp: null                                         creationTimestamp: null
      labels:                                                         labels:
        app: vote-bot                                                   app: vote-bot
                                                              >         linkerd.io/control-plane-ns: linkerd
                                                              >         linkerd.io/proxy-deployment: vote-bot
    spec:                                                           spec:
      containers:                                                     containers:
      - command:                                                      - command:
        - emojivoto-vote-bot                                            - emojivoto-vote-bot
        env:                                                            env:
        - name: WEB_HOST                                                - name: WEB_HOST
          value: web-svc.emojivoto:80                                     value: web-svc.emojivoto:80
        image: buoyantio/emojivoto-web:v5                               image: buoyantio/emojivoto-web:v5
        name: vote-bot                                                  name: vote-bot
        resources: {}                                                   resources: {}
                                                              >       - env:
                                                              >         - name: LINKERD2_PROXY_LOG
                                                              >           value: warn,linkerd2_proxy=info
                                                              >         - name: LINKERD2_PROXY_BIND_TIMEOUT
                                                              >           value: 10s
                                                              >         - name: LINKERD2_PROXY_CONTROL_URL
                                                              >           value: tcp://proxy-api.linkerd.svc.cluster.local:80
                                                              >         - name: LINKERD2_PROXY_CONTROL_LISTENER
                                                              >           value: tcp://0.0.0.0:4190
                                                              >         - name: LINKERD2_PROXY_METRICS_LISTENER
                                                              >           value: tcp://0.0.0.0:4191
                                                              >         - name: LINKERD2_PROXY_PRIVATE_LISTENER
                                                              >           value: tcp://127.0.0.1:4140
                                                              >         - name: LINKERD2_PROXY_PUBLIC_LISTENER
                                                              >           value: tcp://0.0.0.0:4143
                                                              >         - name: LINKERD2_PROXY_POD_NAMESPACE
                                                              >           valueFrom:
                                                              >             fieldRef:
                                                              >               fieldPath: metadata.namespace
                                                              >         image: gcr.io/linkerd-io/proxy:v18.8.4
                                                              >         imagePullPolicy: IfNotPresent
                                                              >         livenessProbe:
                                                              >           httpGet:
                                                              >             path: /metrics
                                                              >             port: 4191
                                                              >           initialDelaySeconds: 10
                                                              >         name: linkerd-proxy
                                                              >         ports:
                                                              >         - containerPort: 4143
                                                              >           name: linkerd-proxy
                                                              >         - containerPort: 4191
                                                              >           name: linkerd-metrics
                                                              >         readinessProbe:
                                                              >           httpGet:
                                                              >             path: /metrics
                                                              >             port: 4191
                                                              >           initialDelaySeconds: 10
                                                              >         resources: {}
                                                              >         securityContext:
                                                              >           runAsUser: 2102
                                                              >         terminationMessagePolicy: FallbackToLogsOnError
                                                              >       initContainers:
                                                              >       - args:
                                                              >         - --incoming-proxy-port
                                                              >         - "4143"
                                                              >         - --outgoing-proxy-port
                                                              >         - "4140"
                                                              >         - --proxy-uid
                                                              >         - "2102"
                                                              >         - --inbound-ports-to-ignore
                                                              >         - 4190,4191
                                                              >         image: gcr.io/linkerd-io/proxy-init:v18.8.4
                                                              >         imagePullPolicy: IfNotPresent
                                                              >         name: linkerd-init
                                                              >         resources: {}
                                                              >         securityContext:
                                                              >           capabilities:
                                                              >             add:
                                                              >             - NET_ADMIN
                                                              >           privileged: false
                                                              >         terminationMessagePolicy: FallbackToLogsOnError
status: {}                                                      status: {}
---                                                             ---
```