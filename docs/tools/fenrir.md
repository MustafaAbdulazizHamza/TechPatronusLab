# Cerberus

=== "Description"
    Fenrir is a brute-forcing tool designed specifically to target the Pandora service as part of this project. It operates by sending HTTPS requests with credentials sourced from a dictionary. The tool halts either upon successful authentication with valid credentials or after exhausting all entries in the wordlist without success.

=== "Code"
    ```go linenums="1" title="main.go"
    package main
    import (
        "bufio"
        "bytes"
        "crypto/tls"
        "flag"
        "fmt"
        "log"
        "net/http"
        "os"
        "strings"
        "time"
    )
    func main() {
        target := flag.String("t", "", "Target host (IP or domain)")
        port := flag.Int("s", 0, "Port number for the Pandora server")
        users := flag.String("u", "", "File path to the list of usernames")
        passwords := flag.String("p", "", "File path to the password dictionary")
        flag.Parse()
        if *target == "" || *port == 0 || *users == "" || *passwords == "" {
            fmt.Println("Error: Missing required flags.")
            flag.Usage()
            os.Exit(1)
        }
        userDict, err := os.Open(*users)
        if err != nil {
            log.Fatal(err)
        }
        defer userDict.Close()
        userScanner := bufio.NewScanner(userDict)
        passwordsDict, err := os.Open(*passwords)
        if err != nil {
            log.Fatal(err)
        }
        defer passwordsDict.Close()
        body := []byte(`"secret-id":"secretID"`)
        for userScanner.Scan() {
            user := strings.TrimSpace(userScanner.Text())
            passwordsDict.Seek(0, 0)
            passwordScanner := bufio.NewScanner(passwordsDict)
            for passwordScanner.Scan() {
                password := strings.TrimSpace(passwordScanner.Text())
                re, err := sendHTTPRequest("GET", fmt.Sprintf("https://%s:%d/secret", *target, *port), user, password, string(body))
                if err == nil {
                    if re.StatusCode != 401 && re.StatusCode != 500 {
                        log.Printf("Account Found: Username: %s, Password: %s (Success)\n", user, password)
                        break
                    } else {
                        log.Printf("Account Check - Username: %s, Password: %s\n", user, password)
                    }
                }
            }
        }
    }
    ```
=== "Help"
    ```
    ./Fenrir [options]
    Options
    -p string
        Specifies the file path to the password dictionary.
    -s int
        Defines the port number for the Pandora server.
    -t string
        Specifies the target host (IP address or domain).
    -u string
        Provides the file path to the list of usernames.
    ```
=== "Download"
    [![GitHub](https://img.shields.io/badge/Source-GitHub-blue)](https://github.com/MustafaAbdulazizHamza/packet-lab/tree/master/Golang/Fenrir)