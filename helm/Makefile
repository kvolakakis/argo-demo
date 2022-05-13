workflows-helm: 
	## inside argo-helm/charts
	helm install argo-workflows ./argo-workflows ## or helm install argo-workflows argo/argo-workflows 
	
workflows-credentials:
	## get credentials
	SECRET=$(kubectl get sa argo-workflows-server -o=jsonpath='{.secrets[0].name}')
	ARGO_TOKEN="Bearer $(kubectl get secret $SECRET -o=jsonpath='{.data.token}' | base64 --decode)"
	echo $ARGO_TOKEN

workflows-publish:
	## port-forward
	kubectl port-forward deployment/argo-workflows-server 2746:2746

workflows-submit-hello:
	## inside folder with 'argo cli'
	argo submit https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml


cd-helm: 
	## inside argo-helm/charts
	helm install argo-cd ./argo-cd ## or helm install argo-workflows argo/argo-workflows 
	
cd-publish:
	## port-forward
	kubectl port-forward service/argo-cd-argocd-server -n default 8080:443

cd-credentials:
	kubectl -n default get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d #get
	./argocd login 127.0.0.1:8080 --username admin --password {TOKEN/password}

cd-submit-hello:
	## inside folder with 'argocd cli'
	./argocd app create guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path guestbook --dest-server https://kubernetes.default.svc --dest-namespace default




