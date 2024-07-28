
{{ define "livenessProbe" }}
livenessProbe:
          failureThreshold: 3
          httpGet:
            path: {{ .Values.ingress.healthCheckPath }}
            port: {{ .Values.containerPort }}
            scheme: HTTP
          initialDelaySeconds: 160
          periodSeconds: 35
          successThreshold: 1
          timeoutSeconds: 30
{{- end }}

{{ define "readinessProbe" }}
readinessProbe:
          failureThreshold: 3
          httpGet:
            path: {{ .Values.ingress.healthCheckPath }}
            port: {{ .Values.containerPort }}
            scheme: HTTP
          initialDelaySeconds: 160
          periodSeconds: 35
          successThreshold: 1
          timeoutSeconds: 30
{{- end }}
