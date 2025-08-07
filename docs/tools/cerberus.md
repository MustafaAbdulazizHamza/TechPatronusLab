# Cerberus

=== "Description"
    Cerberus is a Golang-based port scanner designed to detect a running Pandora server during the reconnaissance phase. The tool performs a full TCP scan, followed by an HTTPS probe to check for the presence of a valid https://IP:Port/secret endpoint. If such an endpoint is discovered, it confirms that Pandora is running on the associated port.


=== "Code"
    ``` go linenums="1" title="main.go"
    package main
    import (
        "flag"
        "fmt"
        "os"
        "strings"
    )
    var (
        ports  = make(chan int, 400)
        result = make(chan string, 400)
    )
    func main() {
        target := flag.String("t", "", "The target IP address or hostname")
        ports := flag.String("p", "", "The port(s) to scan, separated by commas")
        PandoraScan := flag.Bool("v", false, "Enable Pandora-specific scanning mode")
        flag.Parse()
        if *target == "" || *ports == "" {
            fmt.Println("Error: Missing required flags.")
            flag.Usage()
            os.Exit(1)
        }
        ports_range := getPortsRange(*ports)
        ts := strings.Split(*target, ",")
        for _, target := range ts {
            scan(target, ports_range, *PandoraScan)
        }
    }
    ```

=== "Help"
    ```
    ./Cerberus [options]
    Options
    -p string
        Specifies the port(s) to scan, separated by commas.
    -t string
        Specifies the target IP address or hostname.
    -v
        Enables Pandora-specific scanning mode.
    ```
=== "Download"
    [![GitHub](https://img.shields.io/badge/Source-GitHub-blue)](https://github.com/MustafaAbdulazizHamza/packet-lab/tree/master/Golang/Cerberus)
