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



svc

~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service1 # TODO: specify your service name
  labels:
    app: nginx-svc
spec:
  selector:
    app: nginx # TODO: change label selector to match your backend pod
  ports:
  - protocol: TCP
    name: http
    port: 80 # Expose the service on port 80 internally
    targetPort: 80 # Forward traffic to the container's port 80
  type: LoadBalancer # Use LoadBalancer instead of NodePort
~~~

---

~~~yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: '1'
  creationTimestamp: '2025-03-10T14:19:28Z'
  generation: 1
  labels:
    app: nginx
  managedFields:
    - apiVersion: apps/v1
      fieldsType: FieldsV1
      fieldsV1:
        'f:metadata':
          'f:labels':
            .: {}
            'f:app': {}
        'f:spec':
          'f:progressDeadlineSeconds': {}
          'f:replicas': {}
          'f:revisionHistoryLimit': {}
          'f:selector': {}
          'f:strategy':
            'f:rollingUpdate':
              .: {}
              'f:maxSurge': {}
              'f:maxUnavailable': {}
            'f:type': {}
          'f:template':
            'f:metadata':
              'f:labels':
                .: {}
                'f:alibabacloud.com/compute-class': {}
                'f:alibabacloud.com/compute-qos': {}
                'f:app': {}
            'f:spec':
              'f:containers':
                'k:{"name":"nginx"}':
                  .: {}
                  'f:image': {}
                  'f:imagePullPolicy': {}
                  'f:name': {}
                  'f:ports':
                    .: {}
                    'k:{"containerPort":80,"protocol":"TCP"}':
                      .: {}
                      'f:containerPort': {}
                      'f:protocol': {}
                  'f:resources':
                    .: {}
                    'f:limits':
                      .: {}
                      'f:cpu': {}
                  'f:terminationMessagePath': {}
                  'f:terminationMessagePolicy': {}
              'f:dnsPolicy': {}
              'f:restartPolicy': {}
              'f:schedulerName': {}
              'f:securityContext': {}
              'f:terminationGracePeriodSeconds': {}
      manager: okhttp
      operation: Update
      time: '2025-03-10T14:19:28Z'
    - apiVersion: apps/v1
      fieldsType: FieldsV1
      fieldsV1:
        'f:metadata':
          'f:annotations':
            .: {}
            'f:deployment.kubernetes.io/revision': {}
        'f:status':
          'f:availableReplicas': {}
          'f:conditions':
            .: {}
            'k:{"type":"Available"}':
              .: {}
              'f:lastTransitionTime': {}
              'f:lastUpdateTime': {}
              'f:message': {}
              'f:reason': {}
              'f:status': {}
              'f:type': {}
            'k:{"type":"Progressing"}':
              .: {}
              'f:lastTransitionTime': {}
              'f:lastUpdateTime': {}
              'f:message': {}
              'f:reason': {}
              'f:status': {}
              'f:type': {}
          'f:observedGeneration': {}
          'f:readyReplicas': {}
          'f:replicas': {}
          'f:updatedReplicas': {}
      manager: kube-controller-manager
      operation: Update
      subresource: status
      time: '2025-03-10T14:20:31Z'
  name: nginx-deployment-basic
  namespace: nginx
  resourceVersion: '23076'
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
      labels:
        alibabacloud.com/compute-class: general-purpose
        alibabacloud.com/compute-qos: default
        app: nginx
    spec:
      containers:
        - image: >-
            anolis-registry.cn-zhangjiakou.cr.aliyuncs.com/openanolis/nginx:1.14.1-8.6
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
    - lastTransitionTime: '2025-03-10T14:20:31Z'
      lastUpdateTime: '2025-03-10T14:20:31Z'
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: 'True'
      type: Available
    - lastTransitionTime: '2025-03-10T14:19:28Z'
      lastUpdateTime: '2025-03-10T14:20:31Z'
      message: >-
        ReplicaSet "nginx-deployment-basic-b8885b4df" has successfully
        progressed.
      reason: NewReplicaSetAvailable
      status: 'True'
      type: Progressing
  observedGeneration: 1
  readyReplicas: 2
  replicas: 2
  updatedReplicas: 2

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



kubectl get deploy nginx-deployment-basic -o yaml | yq eval '.spec.template' -





kubectl get <resource-type> <resource-name> -o yaml --export > original.yaml

kubectl get pod  -o yaml --export > original.yaml



kubectl get pod nginx-deployment-basic-b8885b4df-lk2s9 -o yaml --export





kubectl get deploy nginx-deployment-basic -n nginx -o yaml







~~~yaml
shell@Alicloud:~$ kubectl get deploy windows -n truth-ai-administrative -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "14"
  creationTimestamp: "2024-12-26T07:51:18Z"
  generation: 18
  labels:
    app: windows
  name: windows
  namespace: truth-ai-administrative
  resourceVersion: "84653845"
  uid: 36cf3e02-3dde-4d94-b8d0-ea115065c71e
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: windows
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      annotations:
        redeploy-timestamp: "1735201291395"
      creationTimestamp: null
      labels:
        app: windows
        pod-template-hash: 978f87cbd
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: alibabacloud.com/nodepool-id
                operator: In
                values:
                - npaf6c0d166f4f4cf4af18e6ae4905cfc0
      containers:
      - env:
        - name: VERSION
          value: "10"
        - name: KVM
          value: "N"
        - name: RAM_SIZE
          value: 24G
        - name: CPU_CORES
          value: "8"
        - name: REGION
          value: en-GB
        - name: KEYBOARD
          value: en-GB
        image: registry.cn-hangzhou.aliyuncs.com/truth-ai/windows:4.07
        imagePullPolicy: IfNotPresent
        name: windows
        ports:
        - containerPort: 8006
          name: tcp
          protocol: TCP
        - containerPort: 3389
          name: tcp1
          protocol: TCP
        - containerPort: 3389
          name: udp
          protocol: UDP
        resources:
          limits:
            cpu: "24"
            memory: 65336Mi
          requests:
            cpu: "0"
            memory: "0"
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
          privileged: true
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /storage
          name: volume-1735198761357
        - mountPath: /dev/kvm
          name: volume-1735198761482
        - mountPath: /dev/net/tun
          name: volume-1735198761708
        - mountPath: /etc/localtime
          name: volume-localtime
        - mountPath: /data
          name: volume-1735205233745
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: acr-hzm
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - hostPath:
          path: /mnt/k8s-volumes/truth-administrative/windows
          type: ""
        name: volume-1735198761357
      - hostPath:
          path: /dev/kvm
          type: ""
        name: volume-1735198761482
      - hostPath:
          path: /dev/net/tun
          type: ""
        name: volume-1735198761708
      - hostPath:
          path: /etc/localtime
          type: ""
        name: volume-localtime
      - hostPath:
          path: /mnt/k8s-volumes/truth-administrative/windows/data
          type: ""
        name: volume-1735205233745
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2024-12-26T07:51:18Z"
    lastUpdateTime: "2024-12-31T03:52:20Z"
    message: ReplicaSet "windows-978f87cbd" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  - lastTransitionTime: "2025-03-07T16:05:26Z"
    lastUpdateTime: "2025-03-07T16:05:26Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  observedGeneration: 18
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
~~~

