# Immich + Docker GPU Fix (`could not select device driver "nvidia"`)

If your Immich containers fail to start with errors like:

```text
could not select device driver "nvidia" with capabilities: [[gpu]]
```

or

```text
could not select device driver "nvidia" with capabilities: [[gpu compute video]]
```

this is usually a Docker daemon/runtime issue, not an Immich image issue.

## What was actually wrong

- Docker was running on a daemon/context without NVIDIA runtime support.
- The user running Docker did not have permission to access the native Docker socket (`/var/run/docker.sock`) and was not in the `docker` group.

Even with NVIDIA drivers and `nvidia-ctk` installed, Docker GPU requests fail until both are fixed.

## What fixed it

1. Add your user to the `docker` group:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

2. Switch to the native Docker Engine context:

```bash
docker context use default
```

3. Ensure NVIDIA runtime is configured for Docker:

```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

4. Verify Docker can use the GPU before starting Immich:

```bash
docker --context default run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
```

If this command works, GPU passthrough is fixed.

5. Start Immich:

```bash
docker --context default compose up -d
docker --context default compose ps
```

## Quick sanity checks

- Confirm you are on the right context:

```bash
docker context ls
```

- Confirm Docker sees runtimes:

```bash
docker info | grep -i runtime -A2
```

- Confirm host GPU works outside Docker:

```bash
nvidia-smi
```

## Notes

- If you are on Docker Desktop Linux, GPU support may not behave like native Docker Engine. Use the native daemon (`default` context) for NVIDIA workloads.
- Fix Docker GPU access first, then troubleshoot Immich-specific settings.
