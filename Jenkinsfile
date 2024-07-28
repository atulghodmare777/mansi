{{- define "java-volumes" }}
- name: {{.Values.name}}-tomcat-logs
  emptyDir: {}
- name: {{.Values.name}}-app-logs
  emptyDir: {}
{{- if .Values.secretRequired }}
- name: {{.Values.namespace}}-db-secret
  secret:
    defaultMode: 0666
    secretName: {{.Values.namespace}}-db-secret
{{- end }}
{{- if or (eq .Values.environment "preprod") (eq .Values.environment "prod") }}
- name: prestop-script
  configMap:
        name: {{ .Values.environment }}-prestop-script-config
{{- end }}
{{- end }}
