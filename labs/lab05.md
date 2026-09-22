# Lab 5: Containerizing ML Models with Docker

In this lab, you will train a Wine classifier in one container, serve predictions from another, and use Docker Compose to run them together. You will use a named volume to share the model and a bind mount to view prediction logs on your computer.

Clone the code from [this repository](https://github.com/MarkArmstbowl/mlip-docker-lab-f26) and follow the instructions in its [README](https://github.com/MarkArmstbowl/mlip-docker-lab-f26/blob/main/README.md). Complete the TODOs in the starter files and consider the **Think about it** questions as you work.

Show your work to a TA during the lab to receive credit.

## Deliverables

- [ ] **Training:** Show that the training script ran in a container and saved the model to a shared volume. Explain why Docker is useful for reproducibility and portability in ML training.
- [ ] **Inference:** Serve predictions from the inference container on localhost port 8081 and show the TA `./logs/predictions.log` on your host. Explain what the Dockerfile does for the inference service.
- [ ] **Volume lifecycle:** Attempt to call the inference service health endpoint before and after removing the Compose named volume. Explain the difference between named volumes and bind mounts in Docker.

## References

- [Install Docker](https://docs.docker.com/get-started/get-docker/)
- [Docker volumes](https://docs.docker.com/engine/storage/volumes/)
- [Docker bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
- [Docker Compose startup order](https://docs.docker.com/compose/how-tos/startup-order/)

## Troubleshooting

- Check that Docker is running and that port 8081 is available.
- If a service fails during startup, compare the training and inference logs with `docker compose logs`.
- If the model is missing, compare the volume names in your commands or Compose file and check them with `docker volume ls`.
- If prediction logs do not appear on your host, check the bind mount path.

See the starter repository's [Troubleshooting section](https://github.com/MarkArmstbowl/mlip-docker-lab-f26/blob/main/README.md#troubleshooting) for more guidance.
