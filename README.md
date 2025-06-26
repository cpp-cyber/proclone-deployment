### Deployment instructions:

1. Set up working kubernetes cluster with istio-system and istio-ingressgateway
2. replace placeholder values (hosts, secrets, egress hosts, etc.) with actual values for your environment
3. Use existing or create self-signed cert:
```openssl req -x509 -nodes -days 1000 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=[yourdomainname]/O=[yourorganization]"```
4. add cert as secret with:
```kubectl create secret tls -n istio-system secret-cert --cert=tls.crt --key=tls.key```
5. apply all the kubectl configs

