{{ define "java-volume-mount" }}
        - mountPath: /usr/local/tomcat/logs
          name: {{ .Values.name}}-tomcat-logs
        - mountPath: /home/DigitApps/Logs
          name: {{ .Values.name}}-app-logs
{{- if .Values.secretRequired }}
        - mountPath: /etc/digit
          name: {{ .Values.namespace}}-db-secret
{{- end }}
{{- if or (eq .Values.environment "preprod") (eq .Values.environment "prod") }}
        - name: prestop-script
          mountPath: /mnt
{{- end}}
{{- end }}
