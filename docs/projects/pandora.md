# Pandora Project
=== "Description"
    Secret management is a critical aspect of modern DevOps environments, where securely handling authentication tokens and other sensitive credentials is essential.
    To address these needs, we introduce Pandora, a secret management system built using the Golang Gin framework. Pandora is designed to run in a containerized environment and provides an API for seamless user interaction. Any HTTPS client operating from a machine with access to the server can act as a Pandora client.
    Pandora aims to offer an efficient and streamlined solution for securely managing secrets in DevOps workflows.

=== "Code"
    ``` golang title="" linenums="1"
    package main

    import (
        "database/sql"
        "github.com/MustafaAbdulazizHamza/Pandora-Server/APIs"
        "github.com/MustafaAbdulazizHamza/Pandora-Server/Middleware"
        _ "github.com/mattn/go-sqlite3"

        "github.com/gin-gonic/gin"
        "log"
        "os"
    )

    func main() {
        gin.SetMode(gin.ReleaseMode)
        router := gin.Default()
        var (
            address string = ":8080"
            db      *sql.DB
            err     error
        )
        if db, err = sql.Open("sqlite3", "./Pandora.db"); err != nil {
            os.Exit(404)
        }
        defer db.Close()
        if err = db.Ping(); err != nil {
            os.Exit(404)
        }
        router.Use(Middleware.AuthenticateUser(db))

        router.POST("/user", APIs.InsertUser(db))
        router.PATCH("/user", APIs.UpdateUserCredentials(db))
        router.DELETE("/user", APIs.DeleteUser(db))
        router.POST("/secret", APIs.PostSecret(db))
        router.GET("/secret", APIs.GetSecret(db))
        router.PATCH("/secret", APIs.UpdateSecret(db))
        router.DELETE("/secret", APIs.DeleteSecret(db))
        err = router.RunTLS(address, "server.crt", "server.key")
        log.Fatal(err)
    }
    ```
=== "Documentation"
    ## Pandora Secrets Management System
    ### Introduction
    Secret management is a critical aspect of modern DevOps environments, where securely handling authentication tokens and other sensitive credentials is essential.  
    To address these needs, we introduce **Pandora**, a secret management system built using the Golang Gin framework. Pandora is designed to run in a containerized environment and provides an API for seamless user interaction. Any HTTPS client operating from a machine with access to the server can act as a Pandora client.  
    Pandora aims to offer an efficient and streamlined solution for securely managing secrets in DevOps workflows.
    ---
    ### Prerequisites
    1. **Golang Installed**: Install Golang to compile the code.  
    2. **Docker Installed**: Install Docker for containerized deployment.  
    3. **Digital Certificate and Private Key**: Obtain a valid digital certificate and private key to enable secure HTTPS communication.
    ---
    ### Installation Guide
    Follow these steps to set up the Pandora HTTPS server:

    1. **Clone the Repository**: Clone the Pandora repository from GitHub:
    ```shell
    git clone https://github.com/MustafaAbdulazizHamza/Pandora-Server.git
    ```
    2. **Compile the Project**: Compile the project so that it can be run:
    ```shell
    go build -o Pandora
    ```
    3.	**Add Digital Certificate and Private Key**: Place your digital certificate and private key files (named server.*) in the project directory.
    4.	**Build Pandora Docker Image**: Create a custom Docker image for Pandora using the following command:
    ```shell
    docker build -t pandora:v1 .
    ```
    4.	**Create a Database Storage Volume**: Set up a volume for the Pandora database using:
    ```shell
    docker volume create pandora-data
    ```
    6. **Run Pandora Container**: Launch the Pandora container and attach the previously created volume:
    ```shell
    docker run -d -p 8080:8080 --name pandora -v pandora-data:/root/ pandora:v1  
    ```
    ---
    ### Notes:
    1. The default root password is root.
    2. You can use Pandora-CLI for server configuration or service acquisition.
    3. Several clients have been developed for this server, such as Pandora-CLI and Go-Pandora.
    ---
    ### Related Tools
    1. **Pandora-Cli**: An HTTPS client CLI tool for simple user interaction with the Pandora Secrets Management System.
    [Pandora-CLI](../tools/pandora-cli.md)
    2. **Go-Pandora**: A Golang library that provides a client interface for using and managing the Pandora Secrets Management System was publish at this repository:
    [Go-Pandora](https://github.com/MustafaAbdulazizHamza/go-pandora)

=== "Download"
    [![GitHub](https://img.shields.io/badge/Source-GitHub-blue)](https://github.com/MustafaAbdulazizHamza/Pandora-Server)

