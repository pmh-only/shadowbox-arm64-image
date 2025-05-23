# 📦 `shadowbox-arm64-image`
Automatically build ARM64 container image for Outline VPN server (shadowbox)

## K8s deployment example
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vpn
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vpn
  template:
    metadata:
      labels:
        app: vpn
    spec:
      containers:
        - name: vpn
          image: ghcr.io/pmh-only/shadowbox:latest
          ports:
            - containerPort: 8080
              protocol: TCP
            - containerPort: 80
              protocol: TCP
          env:
            - name: SB_API_PORT
              value: "80"
            - name: SB_API_PREFIX
              valueFrom:
                secretKeyRef:
                  name: vpn-secret
                  key: SB_API_PREFIX
            - name: SB_CERTIFICATE_FILE
              value: /tmp/shadowbox.crt
            - name: SB_PRIVATE_KEY_FILE
              value: /tmp/shadowbox.key
          volumeMounts:
            - name: vpn-data
              mountPath: /opt/outline
            - name: vpn-data
              mountPath: /root/shadowbox
            - name: vpn-tls
              readOnly: true
              mountPath: /tmp/shadowbox.crt
              subPath: shadowbox.crt
            - name: vpn-tls
              readOnly: true
              mountPath: /tmp/shadowbox.key
              subPath: shadowbox.key
      volumes:
        - name: vpn-data
          persistentVolumeClaim:
            claimName: vpn-pvc
        - name: vpn-tls
          secret:
            secretName: vpn-tls
            items:
              - key: tls.crt
                path: shadowbox.crt
              - key: tls.key
                path: shadowbox.key
            defaultMode: 420
```
