# Розгортання ArgoCD

## Запуск кластера

```sh
k3d cluster create -a 3 mycluster 
```

## Створення Namespace

```sh
kubectl create namespace argocd

namespace/argocd created
```

```sh
kubectl config set-context --current --namespace=argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## Перевірка розгортання ArgoCD

```sh
k get po -n argocd -w
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-redis-b5d6bf5f5-rg8rs                        1/1     Running   0          20m
argocd-notifications-controller-589b479947-zmpqc    1/1     Running   0          20m
argocd-server-7459448d56-9wsdp                      1/1     Running   0          20m
argocd-applicationset-controller-6f9d6cfd58-9vsfj   1/1     Running   0          20m
argocd-repo-server-7b75b45897-fl2gx                 1/1     Running   0          20m
argocd-application-controller-0                     1/1     Running   0          20m
argocd-dex-server-6df5d4f8c4-6rslh                  1/1     Running   0          20m

^C%
```

Розгортання може тривати певний час. Останній раз це тривало десь 9хв.

## Зміна типу служби argocd-server на LoadBalancer:

```sh
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

## Прокидання порту доступу

```sh
➜ kubectl port-forward svc/argocd-server -n argocd 8080:443
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
^Z
[1]  + 80466 suspended  kubectl port-forward svc/argocd-server -n argocd 8080:443

✦ ➜ bg %kubectl
[1]  + 80466 continued  kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Відкриваємо посилання   <https://127.0.0.1:8080> для доступу до вебінтерфейсу

## Вхід та отримання пароля адміністратора

Логін:  `admin`

Для отримання пароля в терміналі виконати команду, яка виведе декодований пароль адміністратора, який потрібно буде вказати у вебінтерфейсі.

```sh
✦ ➜ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath=" {.data.password}"|base64 -d; echo
```

Також для отримання пароля можна скористатись CLI

```log
argocd admin initial-password -n argocd
xxxxxxxxxxxxxxxx

 This password must be only used for first time login. We strongly recommend you update the password using `argocd account update-password`.
```

Для цього його потрібно спочатку встановити CLI

```sh
brew install argocd
```

Докладніше про інші варіанти встановлення CLI – <https://argo-cd.readthedocs.io/en/stable/cli_installation/>

Докладна документація – <https://argo-cd.readthedocs.io/en/stable/>

