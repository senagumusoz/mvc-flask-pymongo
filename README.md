# DevOps Case Study - Flask MongoDB Todo API

## Proje Hakkında

Bu proje, MongoDB veritabanını kullanan Flask tabanlı bir REST API uygulamasıdır.

Case çalışması kapsamında aşağıdaki teknolojiler kullanılmıştır:

* Python 3.12
* Flask
* MongoDB
* Docker
* Docker Compose
* Kubernetes (kind)
* Azure DevOps CI Pipeline

---

# Proje Yapısı

```text
.
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── src/
│   ├── app.py
│   ├── db_config.json
│   ├── factory/
│   └── models/
└── hepapi_k8s/
    ├── mongo-deployment.yaml
    ├── mongo-service.yaml
    ├── app-deployment.yaml
    └── app-service.yaml
```

---

# API Endpointleri

| Metot  | Endpoint    | Açıklama            |
| ------ | ----------- | ------------------- |
| GET    | /todos/     | Tüm kayıtları getir |
| GET    | /todos/{id} | Tek kayıt getir     |
| POST   | /todos/     | Yeni kayıt oluştur  |
| PUT    | /todos/{id} | Kayıt güncelle      |
| DELETE | /todos/{id} | Kayıt sil           |

---

# Uygulamanın Lokal Çalıştırılması

Virtual environment oluşturulur:

```bash
python3 -m venv venv
source venv/bin/activate
```

Gerekli paketler yüklenir:

```bash
python -m pip install -r requirements.txt
```

Uygulama çalıştırılır:

```bash
cd src
python app.py
```

---

# Docker Kullanımı

Docker image oluşturulur:

```bash
docker build -t mvc-flask-pymongo:latest .
```

Container çalıştırılır:

```bash
docker run -d \
--name flask-app \
-p 5001:5000 \
mvc-flask-pymongo:latest
```

---

# Docker Compose

Flask uygulaması ve MongoDB aynı anda ayağa kaldırılır:

```bash
docker compose up -d --build
```

Container durumları kontrol edilir:

```bash
docker compose ps
```

API testi:

```bash
curl http://localhost:5001/todos/
```

Beklenen çıktı:

```json
[]
```

Containerlar durdurulur:

```bash
docker compose down
```

---

# Docker Compose Yapısı

Docker Compose içerisinde:

* Flask Application Container
* MongoDB Container
* MongoDB Health Check

tanımlanmıştır.

MongoDB hazır olmadan Flask uygulamasının başlamaması sağlanmıştır.

```yaml
depends_on:
  mongo:
    condition: service_healthy
```

Bu sayede bağlantı hatalarının önüne geçilmiştir.

---

# Kubernetes Deployment

Local Kubernetes ortamı için Kind kullanılmıştır.

Cluster oluşturma:

```bash
kind create cluster --name hepapi-case
```

Docker image'ın cluster'a yüklenmesi:

```bash
kind load docker-image mvc-flask-pymongo:latest --name hepapi-case
```

Manifest dosyalarının uygulanması:

```bash
kubectl apply -f hepapi_k8s/
```

Pod kontrolü:

```bash
kubectl get pods
```

Beklenen çıktı:

```text
flask-app   Running
mongo       Running
```

Servis kontrolü:

```bash
kubectl get svc
```

Port yönlendirme:

```bash
kubectl port-forward svc/flask-app-service 8077:5000
```

API testi:

```bash
curl http://localhost:8077/todos/
```

Beklenen çıktı:

```json
[]
```

---

# Kubernetes Bileşenleri

MongoDB:

* mongo-deployment.yaml
* mongo-service.yaml

Flask Uygulaması:

* app-deployment.yaml
* app-service.yaml

---

# Azure DevOps CI Pipeline

CI süreci Azure DevOps Classic Editor kullanılarak hazırlanmıştır.

Pipeline akışı:

1. Repository kaynak kodunun alınması
2. Docker image oluşturulması
3. Docker Compose ile servislerin ayağa kaldırılması
4. Uygulamanın hazır olmasının beklenmesi
5. API endpoint testinin gerçekleştirilmesi
6. Ortamın temizlenmesi

Kullanılan test scripti:

```bash
docker compose up -d --build

docker compose ps

sleep 60

docker compose logs app --tail=50

curl --retry 10 \
     --retry-delay 5 \
     --retry-connrefused \
     --fail \
     http://localhost:5001/todos/

docker compose down
```

Pipeline'ın başarılı olması aşağıdaki koşulların sağlandığını göstermektedir:

* Docker image başarıyla oluşturulmuştur.
* Flask uygulaması ayağa kalkmıştır.
* MongoDB başarıyla çalışmaktadır.
* Flask ve MongoDB arasında bağlantı kurulmuştur.
* `/todos/` endpoint'i başarılı şekilde cevap vermektedir.

---

# Karşılaşılan Problemler ve Çözümler

## MongoDB Connection Refused

İlk aşamada:

```text
localhost:27017 connection refused
```

hatası alınmıştır.

Sebep:

Docker içerisinde localhost adresi MongoDB containerını değil Flask containerını göstermektedir.

Çözüm:

```json
mongodb://mongo:27017/
```

adresine geçilmiştir.

---

## MongoDB Timeout Hatası

İlk Docker Compose testlerinde:

```text
ServerSelectionTimeoutError
```

hatası alınmıştır.

Sebep:

MongoDB tamamen ayağa kalkmadan Flask uygulamasının başlaması.

Çözüm:

MongoDB healthcheck tanımlanmış ve Flask uygulamasının MongoDB hazır olana kadar beklemesi sağlanmıştır.

---

## Kubernetes ImagePullBackOff

Kubernetes ortamında MongoDB podu başlangıçta:

```text
ImagePullBackOff
```

durumuna düşmüştür.

Sebep:

Kind cluster'ın Docker Hub üzerinden image çekememesi.

Çözüm:

Image erişim problemi giderilmiş ve pod yeniden oluşturularak servis başarıyla çalıştırılmıştır.

---

# Sonuç

Bu çalışma kapsamında:

* Flask uygulaması Dockerize edilmiştir.
* Docker Compose ile çoklu container mimarisi oluşturulmuştur.
* Kubernetes üzerinde deployment gerçekleştirilmiştir.
* Azure DevOps üzerinde CI Pipeline hazırlanmıştır.
* Uygulama ve veritabanı entegrasyonu başarıyla test edilmiştir.
