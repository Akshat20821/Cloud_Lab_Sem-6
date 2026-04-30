\# exp-11

\# Experiment 11: Orchestration using Docker Compose \& Docker Swarm



\*\*Name:\*\* Akshat Gupta

\*\*Course:\*\* Containerization and DevOps



\## Objective



Understand container orchestration by moving from Docker Compose to Docker Swarm. Learn how to deploy a stack, scale services, and observe self-healing in a Docker Swarm cluster using WordPress and MySQL setup.





\## Prerequisites



\- Docker installed with Swarm mode enabled

\- The `docker-compose.yml` file from Experiment 6 (WordPress + MySQL)







\## PART B – PRACTICAL (EXTENSION OF EXPERIMENT 6)







\### Task 1: Initialize Docker Swarm



Swarm mode turns your current machine into a manager node of a cluster.



```bash

docker swarm init

```



Verify that your node is ready as a Swarm manager:



```bash

docker node ls

```

<img width="982" height="63" alt="image" src="https://github.com/user-attachments/assets/f6bd26e2-3b88-4ad2-8912-d43ece38b308" />









\### Task 3: Deploy as a Stack (Not Just Compose)



In Swarm, we deploy a stack using the same Compose file. Swarm reads the file and creates services, which manage the containers automatically.



```bash

docker stack deploy -c docker-compose.yml wpstack

```

<img width="853" height="87" alt="image" src="https://github.com/user-attachments/assets/44972cfa-60d5-4e40-91c8-2a5c59203715" />







\### Task 4: Verify the Deployment



List all services in the stack:



```bash

docker service ls

```

<img width="983" height="65" alt="image" src="https://github.com/user-attachments/assets/c683fc38-94aa-445d-9ee6-902dba062012" />



See detailed tasks (containers) for a specific service:



```bash

docker service ps wpstack\_wordpress

```



See all running containers (notice they are managed by Swarm):



```bash

docker ps

```

<img width="984" height="183" alt="image" src="https://github.com/user-attachments/assets/75c799bd-0715-433a-8f1e-48913a4875a7" />









\### Task 5: Access WordPress



Open your browser and navigate to:

`http://localhost:8081`



You should see the WordPress setup screen.



<img width="1280" height="1045" alt="image" src="https://github.com/user-attachments/assets/a675397d-303a-4f3f-91d9-4373742307ea" />





\---



\### Task 6: Scale the Application (Swarm's Superpower)



Scale WordPress from 1 to 3 replicas using Swarm's orchestration.



```bash

docker service scale wpstack\_wordpress=3

```

<img width="931" height="137" alt="image" src="https://github.com/user-attachments/assets/4c70ac76-94e0-441c-ade4-50ac46e95bb4" />



Verify the scaling:



```bash

docker service ls

docker service ps wpstack\_wordpress

docker ps | grep wordpress

```

<img width="983" height="194" alt="image" src="https://github.com/user-attachments/assets/16c8d239-b269-4e07-a7c4-7d12997c5da7" />





\*Note: Swarm automatically balances traffic among all 3 containers on port 8081 without port conflicts.\*







\### Task 7: Test Self-Healing (Automatic Recovery)



Swarm automatically replaces failed containers. Let's test it:



\*\*Step 1:\*\* Find a WordPress container ID.

```bash

docker ps | grep wordpress

```



\*\*Step 2:\*\* Kill it to simulate a crash (replace `<container-id>`):

```bash

docker kill <container-id>

```

<img width="984" height="75" alt="image" src="https://github.com/user-attachments/assets/5a416950-c931-4f55-a5b9-d0e0710a94e3" />





\*\*Step 3:\*\* Watch Swarm recreate it automatically:

```bash

docker service ps wpstack\_wordpress

docker ps | grep wordpress

```

<img width="983" height="240" alt="image" src="https://github.com/user-attachments/assets/3279b999-c110-4bf2-bd6c-2cf25afe461e" />





\*Notice the killed container is shut down, and a new container is automatically created to maintain 3 replicas.\*







\### Task 8: Remove the Stack



Clean up the deployed stack and verify removal:



```bash

docker stack rm wpstack

docker service ls

docker ps

```

<img width="981" height="251" alt="image" src="https://github.com/user-attachments/assets/4f43ecff-559f-442e-9d61-3e10c5476b24" />







