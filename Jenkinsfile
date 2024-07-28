apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.name }}
  namespace: {{ .Values.namespace }}
  labels:
    app: {{ .Values.app }}
    environment: {{ .Values.environment }}
spec:
  replicas: {{ .Values.replicas }}
  minReadySeconds: 300
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: {{ .Values.app }}
      environment: {{ .Values.environment }}
  template:
    metadata:
      labels:
        app: {{ .Values.app }}
        environment: {{ .Values.environment }}
    spec:
      {{- if eq .Values.environment "uat" }}
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                - key: role
                  operator: In
                  values:
                  - non-abs-spot
      {{- end }}
      containers: 
      - image: "{{ .Values.image }}:{{ .Values.imageTag }}"
{{- if .Values.isLivenessReadiness }}
{{- include "livenessProbe"  . | indent 8}}
{{- include "readinessProbe"  . | indent 8}}
{{- end}}         
        env: {{ if eq .Values.serviceType "JAVA"}}
        {{- range .Values.env_java}}
          - name: {{ .name }}
            value: {{.value }}
        {{- end }}
        {{ else }}
        {{- range .Values.env_ai}}
          - name: {{ .name }}
            value: {{.value }}
        {{- end }}
        {{- end }}
        imagePullPolicy: {{ .Values.imagePullPolicy }}
        name: {{ .Values.name }}
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop:
            - ALL
        ports:
        - containerPort: {{.Values.containerPort}}
          protocol: TCP
        resources:
          limits:
            cpu: {{ .Values.resources.limits.cpu }}
            memory: {{ .Values.resources.limits.memory }}
          requests:
            cpu: {{ .Values.resources.requests.cpu }}
            memory: {{ .Values.resources.requests.memory }}
        volumeMounts: {{- if eq .Values.serviceType "JAVA"}}
{{- include "java-volume-mount"  . | indent 10}}
        {{- else }}
{{- include "ai-volume-mount" . | indent 10}}
        {{- end }}
      volumes: {{- if eq .Values.serviceType "JAVA" }}
{{- include "java-volumes"  . | indent 7}}
      {{- else  }}
{{- include "ai-volumes" . | indent 7}}
      {{- end }}
