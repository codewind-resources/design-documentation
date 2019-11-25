# Deploying codewind

## Installing a remote codewind-pfe

###### Steps for user of cwctl

cwctl called to Install Codewind on a remote cloud installation

`cwctl --insecure install remote  —namespace {your_namespace}  --kadminuser admin --kadminpass <keycloakPassword>  --krealm codewind --kclient codewind --kdevuser developer --kdevpass <userPassword> Internal flow`

The cli has been design to do almost all the steps required to deploy
codewind remotely by chaining together a number of Kubernetes api
commands. This cli will perform the following steps :

	1. Create codewind-keycloak server key
	2. Create codewind-keycloak server certificate
	3. Deploy Codewind Keycloak secrets
	4. Deploy Codewind Keycloak TLS secrets
	5. Deploy Codewind Keycloak Ingress
	6. Create Keycloak realm
	7. Create Keycloak client
	8. Create Keycloak initial user
	9. Update Keycloak user password
	10. Deploy Codewind service
	11. Deploy Codewind Performance Dashboard
	12. Deploy Codewind Gatekeeper resources
	13. Create codewind-gatekeeper server key
	14. Create codewind-gatekeeper server certificate
	15. Deploy Codewind Gatekeeper secrets
	16. Deploy Codewind Gatekeeper session secrets
	17. Deploy Codewind Gatekeeper secrets
	18. Deploy Codewind Gatekeeper session secrets
	19. Deploy Codewind Gatekeeper TLS secrets
	20. Deploy Codewind Gatekeeper deployment
	21. Deploy Codewind Gatekeeper service
	22. Deploy Codewind Gatekeeper Ingress
