apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Values.name }}
  namespace: {{ .Values.namespace }}
  annotations:
    kubernetes.io/ingress.class: alb
    {{- if and (eq .Values.account "general") (eq .Values.environment "prod") (eq .Values.ingress.type "internal")}}
    alb.ingress.kubernetes.io/group.name: {{ .Values.ingress.groupName }}
    {{- else if and (eq .Values.account "general") (eq .Values.environment "preprod") (eq .Values.ingress.type "internal")}}
    alb.ingress.kubernetes.io/group.name: {{ .Values.ingress.groupName }}
    {{- else if and (eq .Values.account "general") (eq .Values.environment "prod") (eq .Values.ingress.type "internet-facing")}}
    alb.ingress.kubernetes.io/group.name: {{ .Values.ingress.groupName }}
    {{- else if and (eq .Values.account "general") (eq .Values.environment "preprod") (eq .Values.ingress.type "internet-facing")}}
    alb.ingress.kubernetes.io/group.name: {{ .Values.ingress.groupName }}  
    {{- else if and (eq .Values.account "general") (eq .Values.environment "uat") }}
    alb.ingress.kubernetes.io/group.name: {{ .Values.ingress.groupName }}
    {{- else if and (eq .Values.account "general") (eq .Values.environment "dev") }}
    alb.ingress.kubernetes.io/group.name: {{ .Values.ingress.groupName }}
    {{- end }}
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/scheme: {{ .Values.ingress.type }}
    alb.ingress.kubernetes.io/backend-protocol: HTTP
    alb.ingress.kubernetes.io/healthcheck-port: 'traffic-port'
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: '35'
    alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '30'
    alb.ingress.kubernetes.io/healthy-threshold-count: '2'
    alb.ingress.kubernetes.io/unhealthy-threshold-count: '4'
    alb.ingress.kubernetes.io/healthcheck-path: {{ .Values.ingress.healthCheckPath }}
    alb.ingress.kubernetes.io/success-codes: '200{{if ne .Values.serviceType "JAVA"}}-499{{end}}'
    #alb.ingress.kubernetes.io/security-groups: app-alb-ingress
    {{- if and (eq .Values.account "general") (eq .Values.environment "prod") (eq .Values.serviceType "AI") }}
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:706330442742:certificate/ecff21e2-b4f3-4fa6-8e29-d2f5ed9fa5b3
    alb.ingress.kubernetes.io/subnets: subnet-0fdb8a0f3c08d0bca,subnet-09dd75d5ed855a8f3,subnet-0b6df2bd482c9690b
    {{- else if and (eq .Values.account "common") (eq .Values.environment "dev") }}
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:716178926137:certificate/44ee4f47-cb2b-4344-b2d0-9bdd03db732a
    alb.ingress.kubernetes.io/subnets: subnet-0e05345d9328048f8,subnet-028444cb9b82cb367,subnet-0646a9fb4b57b0f2e
    {{- else if and (eq .Values.account "common") (eq .Values.environment "uat") }}
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:716178926137:certificate/44ee4f47-cb2b-4344-b2d0-9bdd03db732a
    alb.ingress.kubernetes.io/subnets: subnet-0e05345d9328048f8,subnet-028444cb9b82cb367,subnet-0646a9fb4b57b0f2e
    {{- else if and (eq .Values.account "common") (eq .Values.environment "preprod") }}
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:716178926137:certificate/44ee4f47-cb2b-4344-b2d0-9bdd03db732a
    alb.ingress.kubernetes.io/subnets: subnet-0d112ab2d7addf3cd,subnet-0eb3bb62570f4f0f3,subnet-08f4d04beda15cb0a
    {{- else if and (eq .Values.account "common") (eq .Values.environment "prod") }}
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:716178926137:certificate/44ee4f47-cb2b-4344-b2d0-9bdd03db732a
    alb.ingress.kubernetes.io/subnets: subnet-b35c80db,subnet-07b46f7a5008c7d48,subnet-0f39ad13a4811680b
    {{- else if and (eq .Values.account "general") (eq .Values.environment "prod") }}
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:706330442742:certificate/ecff21e2-b4f3-4fa6-8e29-d2f5ed9fa5b3
    alb.ingress.kubernetes.io/subnets: subnet-0462535286e9e7bc0,subnet-067a90ea3e919c74b,subnet-0884687b0a5481c45
    {{- else if and (eq .Values.account "general") (eq .Values.environment "preprod") }}
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:706330442742:certificate/ecff21e2-b4f3-4fa6-8e29-d2f5ed9fa5b3
    alb.ingress.kubernetes.io/subnets: subnet-08a16dc5d87def559,subnet-0f913659be1997f0f,subnet-02b5d0c452514c21c
    {{- else if and (eq .Values.account "general") (eq .Values.environment "uat") }}
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:731760562474:certificate/1d256091-5fa4-4a0d-8f83-4b6bb1c4c0db
    alb.ingress.kubernetes.io/subnets: subnet-022e6e60555423505,subnet-05678d6d5b46a2414,subnet-0763ed7aae27ebb30
    {{- else if and (eq .Values.account "general") (eq .Values.environment "dev") }}
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:731760562474:certificate/1d256091-5fa4-4a0d-8f83-4b6bb1c4c0db
    alb.ingress.kubernetes.io/subnets: subnet-01e65b06b4b18b513,subnet-0c647b2d55775db82,subnet-0d8444d24e24e2490
    {{- end }}
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ssl-redirect
            port:
              name: use-annotation
  - host: {{ .Values.ingress.host }}
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: {{ .Values.name | quote }}
              port:
                number: 80
