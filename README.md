# Deploy

- oc311 create -f secret.yaml
- oc311 process -f template.yaml -p PROJECT_NAME=$(oc311 project -q) | oc311 create -f -