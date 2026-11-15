# Starter P5 — Layanan API Kualitas Udara (SS-302)

Layanan contoh untuk praktikum container. Data disimpan di memori, jadi hilang saat container dihapus.

| Endpoint | Fungsi |
| --- | --- |
| `GET /health` | Status layanan |
| `POST /readings` | Kirim bacaan sensor `{"sensor_id": "aq-001", "pm25": 72.4}` |
| `GET /readings?sensor_id=aq-001` | Daftar bacaan |
| `GET /alerts` | Bacaan di atas ambang `ALERT_THRESHOLD` |
| `GET /docs` | Dokumentasi API interaktif |

## 1. Build dan jalankan

```bash
docker build -t aq-service:0.1.0 .
docker run -d --name aq -p 8000:8000 -e ALERT_THRESHOLD=50 aq-service:0.1.0
docker ps                      # tunggu kolom STATUS menjadi (healthy)
```

Buka http://localhost:8000/docs, atau uji dengan curl (Linux, macOS, Git Bash):

```bash
curl -X POST localhost:8000/readings -H "Content-Type: application/json" \
     -d '{"sensor_id":"aq-001","pm25":72.4}'
curl localhost:8000/alerts
```

## 2. Periksa

```bash
docker logs aq                 # log ke stdout
docker exec aq whoami          # harus appuser, bukan root
docker images aq-service       # ukuran image
docker history aq-service:0.1.0
```

## 3. Bandingkan dengan Dockerfile buruk

```bash
docker build -f Dockerfile.buruk -t aq-service:buruk .
docker images aq-service
docker run --rm aq-service:buruk whoami
docker inspect --format '{{.Config.Env}}' aq-service:buruk
```

## 4. Pindai dengan Trivy

Tanpa memasang Trivy, jalankan lewat container:

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image --severity HIGH,CRITICAL aq-service:0.1.0
```

Ulangi untuk `aq-service:buruk` dan bandingkan jumlah temuannya. Untuk alat lokal sekali pakai, tag `latest` masih bisa diterima; di pipeline, pin versi Trivy.

## 5. Push ke GitHub Container Registry

Buat personal access token (classic) dengan scope `write:packages`. Jangan pernah meng-commit token.

```bash
export CR_PAT=<token>
echo $CR_PAT | docker login ghcr.io -u <username-github> --password-stdin
TAG=$(git rev-parse --short HEAD)          # atau versi, misalnya 0.1.0
docker tag aq-service:0.1.0 ghcr.io/<owner>/aq-service:$TAG
docker push ghcr.io/<owner>/aq-service:$TAG
```

Package baru di GHCR bersifat private secara default. Atur aksesnya di Package settings bila anggota kelompok perlu menarik image.

## 6. Bersihkan

```bash
docker rm -f aq
```

Tidak bisa memasang Docker di laptop? Buka repository ini di GitHub Codespaces; Docker sudah tersedia di sana.
