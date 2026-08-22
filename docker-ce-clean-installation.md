# Operator's System Configs — Docker CE

## Target

Clean native Docker setup on Ubuntu:

- Docker Engine + CLI
- Docker Compose V2
- Docker Buildx
- containerd
- No Docker Desktop
- No `docker` group membership; use `sudo docker ...`

## 1. Verify Existing Installation

```bash
cat /etc/os-release
uname -a

docker --version 2>/dev/null || true
docker compose version 2>/dev/null || true
docker context ls 2>/dev/null || true

systemctl status docker --no-pager 2>/dev/null || true
systemctl status docker.socket --no-pager 2>/dev/null || true
systemctl status containerd --no-pager 2>/dev/null || true

dpkg -l | grep -Ei 'docker|containerd|runc' || true
```

Inspect existing data **before deleting anything**:

```bash
sudo docker ps -a
sudo docker images -a
sudo docker volume ls
sudo docker network ls

sudo ls -lah /var/lib/docker
sudo ls -lah /var/lib/containerd
ls -lah ~/.docker

env | grep -E '^DOCKER_' || true
groups
```

Check the Docker repository:

```bash
cat /etc/apt/sources.list.d/docker.sources
apt-cache policy docker-ce
```

## 2. Remove Docker Desktop

```bash
sudo systemctl stop docker-desktop 2>/dev/null || true
sudo systemctl disable docker-desktop 2>/dev/null || true
sudo apt purge -y docker-desktop
sudo apt autoremove -y
```

## 3. Stop and Remove Native Docker

```bash
sudo systemctl stop docker.socket
sudo systemctl stop docker
sudo systemctl stop containerd

sudo apt purge -y   docker-ce   docker-ce-cli   containerd.io   docker-buildx-plugin   docker-compose-plugin   docker-ce-rootless-extras

sudo apt autoremove -y
```

## 4. Remove Old Docker Data

**Destructive: only use when existing Docker data is disposable.**

```bash
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
rm -rf ~/.docker
```

Verify:

```bash
sudo ls -ld /var/lib/docker /var/lib/containerd
```

## 5. Reinstall Native Docker

Use the official Docker APT repository. If it is already configured correctly:

```bash
sudo apt update
apt-cache policy docker-ce
```

Install:

```bash
sudo apt install -y   docker-ce   docker-ce-cli   containerd.io   docker-buildx-plugin   docker-compose-plugin
```

Enable Docker:

```bash
sudo systemctl enable --now docker
```

## 6. Verify

```bash
sudo docker version
sudo docker compose version
sudo docker buildx version
sudo docker info
sudo docker context ls
```

Expected:

- Native `default` context
- 0 containers
- 0 images
- `overlayfs`
- cgroup v2 / systemd
- AppArmor + seccomp
- Docker root: `/var/lib/docker`

Check socket:

```bash
ls -l /var/run/docker.sock
```

Normal form:

```text
srw-rw---- 1 root docker ... /var/run/docker.sock
```

Check user access:

```bash
groups
docker ps
sudo docker ps
```

`docker ps` should fail with permission denied if the user is not in the `docker` group. `sudo docker ps` should work.

## 7. Functional Test

```bash
sudo docker run --rm hello-world
sudo docker ps -a
```

The test container should be removed automatically.

Optional cleanup:

```bash
sudo docker rmi hello-world
```

## 8. Reboot Test

```bash
sudo reboot
```

After reboot:

```bash
systemctl is-active docker
sudo docker ps
sudo docker info
```

## Final State

```text
Ubuntu
├── KVM / libvirt
└── Native Docker
    ├── Docker Engine
    ├── Docker CLI
    ├── Docker Compose V2
    ├── Docker Buildx
    └── containerd
```

**Operator rule:** keep the user out of the `docker` group and use `sudo docker ...`.
