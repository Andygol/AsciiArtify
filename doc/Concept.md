# Порівняльний аналіз інструментів для локального розгортання кластерів Kubernetes


## Вступ

У сучасному світі, де розробка та тестування додатків на платформі Kubernetes стає стандартом, важливо вибрати відповідний інструмент для локального розгортання та тестування. Розглянемо три популярні інструменти: Minikube, kind (Kubernetes IN Docker), та k3d.

## Характеристики

### Minikube

- **Підтримувані ОС та архітектури:** Підтримує різні ОС, включаючи Linux, macOS, та Windows.
  
- **Можливість автоматизації:** Обмежена можливість автоматизації порівняно з іншими інструментами.

- **Додаткові функції:** Включає в себе функції, такі як Dashboard для моніторингу та огляду кластера.

### kind (Kubernetes IN Docker)

- **Підтримувані ОС та архітектури:** Широкі можливості, оскільки використовує Docker як базовий шар.

- **Можливість автоматизації:** Легко інтегрується в CI/CD системи для автоматичного тестування.

- **Додаткові функції:** Орієнтований на швидкість та легкість використання. Підтримує багатовузлові (включаючи HA) кластери. Kind підтримує створення збірок релізів з сирців.

### k3d

- **Підтримувані ОС та архітектури:** Підтримує різні ОС, але інтегрований з Docker.

- **Можливість автоматизації:** Легко автоматизується через інтерфейс командного рядка.

- **Додаткові функції:** Забезпечує легке створення та тестування кластерів Kubernetes у Docker-контейнерах. [vscode-k3d](https://github.com/inercia/vscode-k3d/): Розширення VSCode для роботи з кластером k3d у VSCode

## Переваги та Недоліки

| Характеристика | [Minikube](https://minikube.sigs.k8s.io/)| [kind](https://kind.sigs.k8s.io) | [k3d](https://k3d.io/) |
|--|--|--|--|
| Легкість використання | Зручний для початківців, але може бути обмежений | Легко встановлюється і використовується | Простий у використанні та конфігурації |
| Швидкість розгортання | Залежить від обраного гіпервізору, може бути повільним | Швидке розгортання у Docker-контейнерах | Швидке створення та тестування у Docker-контейнерах |
| Стабільність роботи | Стабільний, але може виникати обмеження масштабування | Стабільний та надійний | Стабільний і легко масштабується |
| Документація та спільнота | Добра документація, активна спільнота | Прийнятна документація, сильна спільнота | Задовільна документація, активна спільнота |
| Налаштування та використання | Має багато параметрів, що може бути складним для новачків | Простий у використанні та конфігурації | Легко конфігурується та використовується |
| Масштабування | Тільки один вузол | Це єдине локальне рішення з HA-кластером з кількома control-plane вузлами. | Підтримує кілька кластерів і підтримує кілька робочих вузлів на кластер. Легко зупиняє та запускає кластери без втрати їх стану.
| Використання з Podman | Обмежене (експериментальне використання) | Краща підтримка роботи, робота у rootless режимі | Обмежене (експериментальне використання)


## Демонстрація

[![asciicast](https://asciinema.org/a/gFdJ22LYufkr4gqFpWuEunKST.svg)](https://asciinema.org/a/gFdJ22LYufkr4gqFpWuEunKST)

1. Встановлення k3d:
   ```sh
   curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
   ```

   or
   
   ```sh
   brew install k3d
   ```

### Створення кластера та розгортання застосунку

1. Створення кластера:
   ```sh
   k3d cluster create -a 3 mycluster
   ```

2. Розгортання застосунку "Hello Node":
   ```sh
   ➜ kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
   
   deployment.apps/hello-node created
   ```

3. Перегляд розгортання
   ```sh
   ➜ kubectl get deployments hello-node 

   NAME         READY   UP-TO-DATE   AVAILABLE   AGE
   hello-node   1/1     1            1           85s
   ```

4. Перегляд поду
   ```sh
   ➜ kubectl get po

   NAME                          READY   STATUS    RESTARTS   AGE
   hello-node-7579565d66-ctxhj   1/1     Running   0          2m26s
   ```

5. Перегляд подій
   ```sh
   ➜ kubectl get events
   ```

6. Перегляд конфігурації
   ```sh
   kubectl config view
   ```

7. Перегляд журналу роботи контейнера в поді
   ```sh
   ➜ kubectl logs hello-node-7579565d66-ctxhj 

   I1122 22:58:28.405151       1 log.go:195] Started HTTP server on port 8080
   I1122 22:58:28.405261       1 log.go:195] Started UDP server on port  8081
   ```

### Створення сервісу

1. Експозиція поду в загальнодоступну мережу за допомогою команди kubectl expose:
   ```sh
   ➜ kubectl expose deployment hello-node --type=LoadBalancer --port=8080
   
   service/hello-node exposed
   ```

2. Перевірка створення сервісу:
   ```sh
   ➜ kubectl get services

   NAME         TYPE           CLUSTER-IP     EXTERNAL-IP                                   PORT(S)          AGE
   kubernetes   ClusterIP      10.43.0.1      <none>                                        443/TCP          21m
   hello-node   LoadBalancer   10.43.244.30   172.23.0.3,172.23.0.4,172.23.0.5,172.23.0.6   8080:31515/TCP   4m10s
   ```

3. Port-forward
   ```sh
   ➜ kubectl port-forward deployments/hello-node 8080&
   
   [1] 65587
   Forwarding from 127.0.0.1:8080 -> 8080
   Forwarding from [::1]:8080 -> 8080
   ```

4. Перевірка доступності застосунку
   ```sh
   ✦ ➜ curl localhost:8080

   Handling connection for 8080
   NOW: 2023-11-22 23:32:31.311009595 +0000 UTC m=+2042.926857596%
   ```

Все працює!!!


## Висновки

Після уважного порівняльного аналізу, рекомендуємо використовувати `k3d` для підготовки PoC для AsciiArtify. K3d надає зручний інтерфейс для локального розгортання та тестування Kubernetes кластерів, має швидкість та стабільність, а його документація та активна спільнота роблять його привабливим вибором для стартапу. Також, враховуючи ризики ліцензування Docker, можна розглянути альтернативу у вигляді використання Podman, хоча й робота з Podman все ще працює в експериментальному режимі.