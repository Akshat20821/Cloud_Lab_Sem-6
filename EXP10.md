\# exp10

\# Experiment 10: SonarQube — Static Code Analysis



\*\*Name:\*\* Akshat Gupta

\*\*Course:\*\* Containerization and DevOps



\---



\## Objective



Perform static code analysis on a Java application using SonarQube to automatically detect bugs, security vulnerabilities, and code smells — and understand how it integrates into a CI/CD pipeline.



\---



\## Theory



\### Problem Statement

Code bugs and security issues are often found too late — during testing or even after deployment. Manual code reviews are slow, inconsistent, and don't scale as teams grow.



\### What is SonarQube?

SonarQube is an open-source platform that automatically scans source code for bugs, security vulnerabilities, and maintainability issues — \*\*without running the code\*\*. This is called \*\*static analysis\*\*.



\### Key Terms



| Term | Meaning |

|------|---------|

| \*\*Quality Gate\*\* | A set of rules; code must pass before deployment |

| \*\*Bug\*\* | Code that will likely break or behave incorrectly |

| \*\*Vulnerability\*\* | A security weakness in the code |

| \*\*Code Smell\*\* | Code that works but is poorly written or hard to maintain |

| \*\*Technical Debt\*\* | Estimated time to fix all issues |

| \*\*Coverage\*\* | Percentage of code tested by unit tests |

| \*\*Duplication\*\* | Repeated code blocks (copy-paste) |



\### Lab Architecture



```

┌─────────────────────────────────────────────────────────┐

│                    Your Machine / CI                    │

│                                                         │

│   ┌──────────────┐        ┌──────────────────────────┐  │

│   │  Your Code   │──────▶ │    Sonar Scanner         │  │

│   │  (Java, JS,  │ scans  │  (CLI / Maven / Jenkins) │  │

│   │   Python...) │        └────────────┬─────────────┘  │

│   └──────────────┘                     │ sends report   │

│                                        ▼                │

│                          ┌─────────────────────────┐    │

│                          │   SonarQube Server      │    │

│                          │   (runs on port 9000)   │    │

│                          │   ┌─────────────────┐   │    │

│                          │   │ Analysis Engine │   │    │

│                          │   │ Quality Gates   │   │    │

│                          │   │ Web Dashboard   │   │    │

│                          │   └────────┬────────┘   │    │

│                          └───────────┼─────────────┘    │

│                                      │ stores results   │

│                          ┌───────────▼─────────────┐    │

│                          │   PostgreSQL Database   │    │

│                          └─────────────────────────┘    │

└─────────────────────────────────────────────────────────┘

```



\---



\## Prerequisites



\- Docker Desktop installed and running

\- Maven (`mvn`) installed (or use the Docker-based scanner)

\- Terminal / command line



Check Docker is running:



```bash

docker --version

docker-compose --version

```



\---



\## Step 1: Start the SonarQube Server



Navigate to the experiment folder and start all containers:



```bash

cd \~/Desktop/exp10

docker-compose up -d

```

<img width="1008" height="252" alt="image" src="https://github.com/user-attachments/assets/0bc6b92e-5221-4de6-83c7-a69062cbbc18" />



Watch logs until you see \*\*"SonarQube is operational"\*\*:



```bash

docker-compose logs -f sonarqube

```



<img width="1280" height="1058" alt="image" src="https://github.com/user-attachments/assets/7dbe8b4b-5f5d-463a-9c04-8da65935f275" />





Verify containers are running:



```bash

docker ps

```

<img width="1005" height="18" alt="image" src="https://github.com/user-attachments/assets/4c8322b3-6503-4bc8-9835-fd8ac5e78db3" />



\*\*Expected:\*\* Two containers — `sonarqube` and `sonar-db` — both with status `Up`.





\---



\## Step 2: Open SonarQube Dashboard



Open your browser and go to:



```

http://localhost:9000

```



\- Default credentials: \*\*admin / admin\*\*

\- You will be prompted to change the password on first login (set it to something like `admin123`)



<img width="1013" height="567" alt="image" src="https://github.com/user-attachments/assets/dd8aa26a-758c-4dda-aa2e-8c44e696ac34" />





<img width="1011" height="583" alt="image" src="https://github.com/user-attachments/assets/5a51a11b-05be-4abc-a1a4-0e03b45aeaf4" />





\---



\## Step 3: Generate an Authentication Token



The scanner needs a token to authenticate with the server.



1\. Click your \*\*user icon\*\* (top right) → \*\*My Account\*\*

2\. Click the \*\*"Security"\*\* tab

3\. Under \*\*"Generate Tokens"\*\*, type a name: `scanner-token`

4\. Click \*\*"Generate"\*\*

5\. \*\*Copy the token immediately\*\* — it is shown only once!







<img width="1014" height="666" alt="image" src="https://github.com/user-attachments/assets/8af5ebba-7358-4499-a298-860fdf65869d" />



Now export the token as an environment variable in your terminal (replace with your actual token):



```bash

export SONAR\_TOKEN=sqp\_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

```



\---



\## Step 4: Create the Sample Java Application



The project files are already created under `sample-java-app/`. Here is what each file contains:



\### `src/main/java/com/example/Calculator.java`



A Java class intentionally containing:

\- \*\*Bug:\*\* Division by zero (unhandled)

\- \*\*Code Smell:\*\* Unused variable

\- \*\*Vulnerability:\*\* SQL Injection risk

\- \*\*Code Smell:\*\* Duplicated `multiply()` and `multiplyAlt()` methods

\- \*\*Bug:\*\* Null pointer risk in `getName()`

\- \*\*Code Smell:\*\* Empty catch block in `riskyOperation()`



\### `pom.xml`



Maven build file with the SonarQube Maven plugin (`sonar-maven-plugin 3.9.1.2184`) configured.



\---



\## Step 5: Update the Token in pom.xml



Replace `YOUR\_TOKEN\_HERE` in `pom.xml` with your actual token:



```bash

cd \~/Desktop/exp10/sample-java-app

sed -i '' "s/YOUR\_TOKEN\_HERE/$SONAR\_TOKEN/" pom.xml

```

<img width="1011" height="17" alt="image" src="https://github.com/user-attachments/assets/36ac7348-b8a2-4cdc-9244-e7926dec817b" />



Verify the replacement:



```bash

grep "sonar.login" pom.xml

```



\---



\## Step 6: Run the SonarQube Scan



\### Option A — Maven Plugin (Recommended if Maven is installed)



```bash

cd \~/Desktop/exp10/sample-java-app

mvn sonar:sonar -Dsonar.login=$SONAR\_TOKEN

```



Maven will compile the code and send the analysis report to the SonarQube server.







\### Option B — Docker-based Scanner CLI (No Maven needed)



First, find the exact Docker network name:



```bash

docker network ls | grep sonarqube

```



Then run the scanner container:



```bash

cd \~/Desktop/exp10/sample-java-app



docker run --rm \\

&#x20; --network exp10\_sonarqube-lab \\

&#x20; -e SONAR\_TOKEN="$SONAR\_TOKEN" \\

&#x20; -v "$(pwd):/usr/src" \\

&#x20; sonarsource/sonar-scanner-cli \\

&#x20; -Dsonar.host.url=http://sonarqube:9000 \\

&#x20; -Dsonar.projectBaseDir=/usr/src \\

&#x20; -Dsonar.projectKey=sample-java-app

```



> \*\*Note:\*\* Use `http://sonarqube:9000` (container name), \*\*not\*\* `localhost`, because the scanner container is on the same Docker network as the server.



<img width="1011" height="564" alt="image" src="https://github.com/user-attachments/assets/c3644199-77a1-4c53-9342-fc0a8be333a9" />





\---



\## Step 7: View Results in the Dashboard



After the scan finishes, open:



```

http://localhost:9000/dashboard?id=sample-java-app

```



<img width="1012" height="556" alt="image" src="https://github.com/user-attachments/assets/f2aa9f52-133a-4819-b856-69186683c7e3" />







\---



\## Step 8: Jenkins Integration (CI/CD — Reference)



Once SonarQube is working locally, it can be added to a Jenkins pipeline so every code commit is automatically scanned.



```groovy

// Jenkinsfile

pipeline {

&#x20;   agent any

&#x20;   environment {

&#x20;       SONAR\_HOST\_URL = 'http://sonarqube:9000'

&#x20;       SONAR\_TOKEN = credentials('sonar-token')

&#x20;   }

&#x20;   stages {

&#x20;       stage('Checkout') {

&#x20;           steps { checkout scm }

&#x20;       }

&#x20;       stage('SonarQube Analysis') {

&#x20;           steps {

&#x20;               withSonarQubeEnv('SonarQube') {

&#x20;                   sh 'mvn clean verify sonar:sonar'

&#x20;               }

&#x20;           }

&#x20;       }

&#x20;       stage('Quality Gate') {

&#x20;           steps {

&#x20;               timeout(time: 5, unit: 'MINUTES') {

&#x20;                   waitForQualityGate abortPipeline: true

&#x20;               }

&#x20;           }

&#x20;       }

&#x20;       stage('Build') {

&#x20;           steps { sh 'mvn package' }

&#x20;       }

&#x20;       stage('Deploy') {

&#x20;           steps {

&#x20;               sh 'docker build -t sample-app .'

&#x20;               sh 'docker run -d -p 8080:8080 sample-app'

&#x20;           }

&#x20;       }

&#x20;   }

}

```

<img width="623" height="339" alt="image" src="https://github.com/user-attachments/assets/7d9b3ee4-df55-4ab8-b51e-cb8865622dfb" />



<img width="1013" height="646" alt="image" src="https://github.com/user-attachments/assets/f62c2c22-7192-4a41-a29e-be5073dc4815" />



<img width="1025" height="505" alt="image" src="https://github.com/user-attachments/assets/0998e9fc-1047-4e23-942f-ae64483c7914" />



\*\*Pipeline flow:\*\*



<img width="1012" height="442" alt="image" src="https://github.com/user-attachments/assets/b6f65384-a8cb-4e90-bb22-c41d5750a069" />



\---







\## Tool Comparison Matrix



| Feature | Jenkins | Ansible | Chef | SonarQube |

|---------|---------|---------|------|-----------|

| Primary Purpose | CI/CD Automation | Config Management | Config Management | Code Quality |

| Architecture | Master-Agent | Agentless | Client-Server | Client-Server |

| Language | Java / Groovy | YAML | Ruby | Java |

| Learning Curve | Moderate | Low | High | Low |

| Setup Complexity | Moderate | Simple | Complex | Simple |



\---



\## Key Concepts Summary



\- \*\*SonarQube Server\*\* = The brain — receives, stores, and displays analysis results

\- \*\*Sonar Scanner\*\* = The worker — reads your code and sends the report to the server

\- \*\*Both are required.\*\* The Scanner needs a Token to talk to the Server

\- \*\*Quality Gates\*\* automatically block bad code from being deployed



\---



\## Best Practices



\- \*\*Security:\*\* Never hardcode tokens — use environment variables or a secrets manager

\- \*\*Code Quality:\*\* Set Quality Gates to block merges when coverage drops below 80%

\- \*\*Scan on every pull request\*\*, not just nightly builds

\- \*\*Fix issues as they appear\*\* — do not let technical debt accumulate

\- \*\*Version all config files in Git\*\*



