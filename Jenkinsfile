apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ .Values.name }}
  namespace: {{ .Values.namespace }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ .Values.name }}
  minReplicas: {{ .Values.spec.minReplicas }}
  maxReplicas: {{ .Values.spec.maxReplicas }}
  metrics:
  - type: Resource
    resource:
      name: {{ .Values.spec.metrics.memoryresource.memoryname }}
      #targetAverageUtilization: 70
      target:
        type: Utilization
        averageUtilization: {{ .Values.spec.metrics.memoryresource.memorytarget.memoryaverageUtilization }}
  - type: Resource
    resource:
      name: {{ .Values.spec.metrics.cpuresource.cpuname }}
      #targetAverageUtilization: 60
      target:
        type: Utilization
        averageUtilization: {{ .Values.spec.metrics.cpuresource.cputarget.cpuaverageUtilization }}
