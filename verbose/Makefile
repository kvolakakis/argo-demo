verbose: 
	kubectl create ns argo
	kubectl apply -n argo -f https://raw.githubusercontent.com/argoproj/argo-workflows/master/manifests/quick-start-postgres.yaml

	
credentials:
	## get credentials
	SECRET=$(kubectl get sa -n argo argo-workflows-server -o=jsonpath='{.secrets[0].name}')
	ARGO_TOKEN="Bearer $(kubectl get secret -n argo $SECRET -o=jsonpath='{.data.token}' | base64 --decode)"
	echo $ARGO_TOKEN

publish:
	## port-forward
	kubectl port-forward deployment/argo-server 2746:2746 -n argo

submit-hello:
	## inside folder with 'argo cli'
	argo submit https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml