---
# draft: true 
date: 2025-08-21
categories:
  - DevOps
authors:
  - pawlowiczf
slug: proto-repository

pin: true

links:
  - GCP Compute Engine: https://cloud.google.com/products/compute
---

# Setup remote repository for `.proto` and `gRPC` files 
In a microservices architecture, Protobuf files and generated stubs are often used by different services located in separate repositories. How can we **share** them across applications while keeping them **consistent**? How can we establish a **single source of truth**, streamline modifications, and automate deployment?

<!-- more -->

Project files: [schema-definitions-repository](../files/p2025-08-21-proto-repository/schema-definitions.zip)

# Design proposal
We'll create a single Git repository to store all Proto files, serving as the single source of truth. Next, we'll set up a server using a cloud provider (e.g., AWS or GCP). In these instructions, we use GCP Compute Engine, but any server with a public IP and the ability to run Docker containers would work. 

On that server, we would run a Nexus repository, which stores generated code by Proto compiler. In addition, we will add Github Actions CI to detect changes in Git repository, compile Proto files and push/upload artifacts to Nexus.  

## Configure server using cloud provider 

1. Set up a `Compute Engine` instance in GCP using the Ubuntu operating system. Select an appropriate machine type, with minimum specifications of 2vCPUs, 4 GB RAM, and 10 GB disk space. I've used `e2-medium` machine and it worked fine. Allow HTTP and HTTPS connections. 
2. Set up `Firewall rule` in a different section and assign it to proper network. `Firewall rule` must allow external traffic coming on port `80` (add also `443`).
3. Generate SSH keys using `ssh-keygen` command. It it possible to create an universal keys that would work for every VM instances, but it is more secure to add a key directly to just one VM. Edit VM instance, find SSH section and add public key there. 
4. Connect to instance using SSH and private key (for example with `Termius`) and do following:
- create new directory `nexus` with: `mkdir nexus` and `cd nexus`
- create new file and paste this content: `nano docker-compose.yaml`. 
```

```
- install `Docker` and `docker compose` with [this tutorial](https://docs.docker.com/engine/install/ubuntu/)
- run service `docker compose run -d`

## Create Git repository 

```xml title="pom.xml"
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.repository</groupId>
    <artifactId>protos</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
            <version>4.32.0</version>
        </dependency>

        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-netty-shaded</artifactId>
            <version>1.74.0</version>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-protobuf</artifactId>
            <version>1.74.0</version>
        </dependency>

        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-stub</artifactId>
            <version>1.74.0</version>
        </dependency>

        <dependency>
            <groupId>javax.annotation</groupId>
            <artifactId>javax.annotation-api</artifactId>
            <version>1.3.2</version>
        </dependency>
        
    </dependencies>

    <distributionManagement>
        <snapshotRepository>
            <id>nexus-snapshots</id>
            <url>${secrets.NEXUS_URL}/repository/maven-snapshots/</url>
        </snapshotRepository>
        <repository>
            <id>nexus-releases</id>
            <url>${secrets.NEXUS_URL}/repository/maven-releases/</url>
        </repository>
    </distributionManagement>

    <build>

        <plugins>
        
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>${java.version}</source> 
                    <target>${java.version}</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>3.1.1</version>
            </plugin>

        </plugins>

    </build>
</project>
```

```xml title="pom-validate.xml"
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.repository</groupId>
    <artifactId>protos</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <java.version>21</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
            <version>4.32.0</version>
        </dependency>

        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-netty-shaded</artifactId>
            <version>1.74.0</version>
            <scope>runtime</scope>
        </dependency>

        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-protobuf</artifactId>
            <version>1.74.0</version>
        </dependency>

        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-stub</artifactId>
            <version>1.74.0</version>
        </dependency>

        <dependency>
            <groupId>javax.annotation</groupId>
            <artifactId>javax.annotation-api</artifactId>
            <version>1.3.2</version>
        </dependency>
        
    </dependencies>

    <build>

        <plugins>
        
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>${java.version}</source> 
                    <target>${java.version}</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.3.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                        </manifest>
                    </archive>
                </configuration>
            </plugin>

        </plugins>

    </build>
</project>
```