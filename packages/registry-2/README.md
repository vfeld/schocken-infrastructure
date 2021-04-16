# Here some basic test on whether the pods and services are configured correctly.
# Run below in a container having sufficient privilidges in the ci namespace

apk add jq skopeo
kubectl get secret -n ci docker-auth-tls-certificate -o jsonpath="{.data}" | jq -r '."tls.crt"' | base64 -d > /tmp/docker-auth-tls-certificate.crt
kubectl get secret -n ci  registry-tls-certificate -o jsonpath="{.data}" | jq -r '."tls.crt"' | base64 -d > /tmp/registry-tls-certificate.crt
kubectl get secret -n ci  ci-certificate-authority -o jsonpath="{.data}" | jq -r '."ca.crt"' | base64 -d > /tmp/ci-certificate-authority.crt

curl -vvv --cacert /tmp/registry-tls-certificate.crt https://registry.ci.svc.cluster.local/v2/
curl -vvv --cacert /tmp/docker-auth-tls-certificate.crt -L "https://docker-auth.ci.svc.cluster.local/auth?service=registry.ci.svc.cluster.local&scope=repository:samalba/my-app:push" | jq .

docker pull hello-world
docker tag hello-world registry.ci.svc.cluster.local/vfeld/hello-world
docker push registry.ci.svc.cluster.local/vfeld/hello-world

mkdir -p /tmp/registry/certs
kubectl get secret -n ci  registry-tls-certificate -o jsonpath="{.data}" | jq -r '."ca.crt"' | base64 -d > /tmp/registry/certs/ca.crt
kubectl get secret -n ci  registry-tls-certificate -o jsonpath="{.data}" | jq -r '."tls.crt"' | base64 -d > /tmp/registry/certs/client.cert
kubectl get secret -n ci  registry-tls-certificate -o jsonpath="{.data}" | jq -r '."tls.key"' | base64 -d > /tmp/registry/certs/client.key
GODEBUG=x509ignoreCN=0 skopeo list-tags --cert-dir /tmp/registry/certs docker://registry.ci.svc.cluster.local/vfeld/hello-world
