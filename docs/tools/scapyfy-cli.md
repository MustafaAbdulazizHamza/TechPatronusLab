# TCP_PortScanner

=== "Description"
    **Scapyfy CLI** is a cross-platform command-line interface (CLI) tool designed to be an HTTP client for the **Scapyfy packet crafting agent**. Developed in **Go (Golang)** and utilizing the **Cobra library** for command parsing, it offers a simple, powerful way to interact with the Scapyfy API.
=== "Code"
    ```golang title="main.go" linenums="1"
    package main

    import (
        "fmt"
        "os"
        "github.com/MustafaAbdulazizHamza/Scapyfy-CLI/cmd"
    )

    func main() {
        if err := cmd.Execute(); err != nil {
            fmt.Fprintf(os.Stderr, "Error: %v\n", err)
            os.Exit(1)
        }
    }
    ```
=== "Documentation"
    ## 💻 Scapyfy CLI: Cross-Platform HTTP Client

    **Scapyfy CLI** is a cross-platform command-line interface (CLI) tool designed to be an HTTP client for the **Scapyfy packet crafting agent**. Developed in **Go (Golang)** and utilizing the **Cobra library** for command parsing, it offers a simple, powerful way to interact with the Scapyfy API.


    ## ⚙️ Requirements

    1.  **Go (Golang)**: You must have Go installed on your system.


    ## 🚀 Setting Up

    Follow these steps to set up and build the tool.

    **1. Clone the repository (Same for Linux/macOS/Windows)**
    *(Requires Git to be installed)*
    ```bash
    git clone [https://github.com/MustafaAbdulazizHamza/Scapyfy-CLI.git](https://github.com/MustafaAbdulazizHamza/Scapyfy-CLI.git)
    ````

    **2. Build the tool (Same for Linux/macOS/Windows)**
    *First, navigate to the project directory:*

    ```bash
    cd Scapyfy-CLI
    ```

    *Then, build the executable. This command is the same for Go on all platforms. It creates `Scapyfy-CLI` on Linux/macOS and `Scapyfy-CLI.exe` on Windows.*

    ```bash
    go build
    ```

    **3. Configure Credentials**
    *Find and edit the file named `scapyfy_config.json` in the project's root directory. Define your credentials and the URL of the Scapyfy server.*

    ```json
    {
    "username": "Your username",
    "password": "Password",
    "token": "",
    "base_url": "URL"
    }
    ```


    ## 💡 Basic Usage

    Once built, you can run the executable from the current directory.

    **1. Login**
    *Logs in using the credentials in `scapyfy_config.json`, retrieves the authentication token, and automatically updates the config file.*

    * **Linux / macOS:**
        ```bash
        ./Scapyfy-CLI login
        ```
    * **Windows (Command Prompt):**
        ```bash
        .\Scapyfy-CLI.exe login
        ```

    **2. Active Packet Crafting**
    *Sends the crafted packets and receives a detailed report after analysis.*

    * **Linux / macOS:**
        ```bash
        ./Scapyfy-CLI craft --active --prompt "Your prompt here" --iters <Number of iteration>
        ```
    * **Windows (Command Prompt):**
        ```bash
        .\Scapyfy-CLI.exe craft --active --prompt "Your prompt here" --iters <Number of iteration>
        ```

    **3. Passive Packet Crafting**
    *Returns only the crafted packet in JSON format without sending it for analysis.*

    * **Linux / macOS:**
        ```bash
        ./Scapyfy-CLI craft --passive --prompt "Your prompt here"
        ```
    * **Windows (Command Prompt):**
        ```bash
        .\Scapyfy-CLI.exe craft --passive --prompt "Your prompt here"
        ```


    ### Command Flags Explained

    * **`--active`**: Sends the crafted packets for active analysis after generation.
    * **`--passive`**: Returns the packet structure as JSON without active transmission/analysis.
    * **`--prompt`**: The LLM prompt describing the packet(s) you want to craft (e.g., "Craft an ARP request packet").
    * **`--iters`**: **(Optional)** Specifies the number of iterations the LLM is allowed before being forced to stop (default is 4). This flag is only relevant for the `--active` mode.

    <!-- end list -->

    ## 📚 References

    1.  [**Scapyfy:**](https://mustafaabdulazizhamza.github.io/TechPatronusLab/projects/scapyfy/#__tabbed_1_1)
    2.  **User Guide:** For a complete list of commands and flags, use the `--help` argument.


=== "Download"
    [![GitHub](https://img.shields.io/badge/Source-GitHub-blue)](https://github.com/MustafaAbdulazizHamza/Scapyfy-CLI)