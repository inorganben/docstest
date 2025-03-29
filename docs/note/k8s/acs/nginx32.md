~~~yaml
apiVersion: apps/v1 # for versions before 1.8.0 use apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment-basic
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        alibabacloud.com/compute-class: general-purpose
        alibabacloud.com/compute-qos: default
    spec:
    #  nodeSelector:
    #    env: test-team
      containers:
      - name: nginx
        image: anolis-registry.cn-zhangjiakou.cr.aliyuncs.com/openanolis/nginx:1.14.1-8.6 # replace it with your exactly <image_name:tags>
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: "500m"
~~~

~~~yaml
 kubectl get deploy nginx-deployment-basic -n nginx -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2025-03-10T14:19:28Z"
  generation: 1
  labels:
    app: nginx
  name: nginx-deployment-basic
  namespace: nginx
  resourceVersion: "23076"
  uid: d32d25f0-367b-4fb1-9004-d27c454f7e1b
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        alibabacloud.com/compute-class: general-purpose
        alibabacloud.com/compute-qos: default
        app: nginx
    spec:
      containers:
      - image: anolis-registry.cn-zhangjiakou.cr.aliyuncs.com/openanolis/nginx:1.14.1-8.6
        imagePullPolicy: IfNotPresent
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
        resources:
          limits:
            cpu: 500m
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: "2025-03-10T14:20:31Z"
    lastUpdateTime: "2025-03-10T14:20:31Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2025-03-10T14:19:28Z"
    lastUpdateTime: "2025-03-10T14:20:31Z"
    message: ReplicaSet "nginx-deployment-basic-b8885b4df" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 2
  replicas: 2
  updatedReplicas: 2
~~~

