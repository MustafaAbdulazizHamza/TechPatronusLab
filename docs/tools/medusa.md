# Medusa

=== "Description"
    Medusa is a Python wrapper for the Medusa brute-force tool. It runs Medusa against a host using provided usernames, passwords, module, and port, captures successful logins, and saves results as JSON.
=== "Code"
    ``` python title="medusa.py" linenums="1"
    import subprocess
    import os
    import sys
    import json
    class Medusa:
        def __init__(self, usernames:str, wordlist:str, module:str, port: int) -> None:
            if not os.path.isfile(usernames):
                print("The usernames file was not found!")
                sys.exit(404)
            if not os.path.isfile(wordlist):
                print("The wordlist file was not found!")
                sys.exit(404)
            self._usernames = usernames
            self._wordlist = wordlist
            self._module = module
            self.port = port
            self.accounts = []
        def Run(self, host:str) -> None:
            p = subprocess.run(f"medusa -h {host} -M {self._module} -U {self._usernames} -P {self._wordlist} -n {self.port}", shell=True, capture_output=True, text=True)
            if p.returncode != 0:
                print(p.stderr)
                sys.exit(1)
            out = [o for o in p.stdout.split("\n") if "[SUCCESS]" in o]
            self.accounts[host] = [(out[i].split()[6], out[i].split()[8]) for i in range(len(out))]
        def As_JSON(self, output: str) -> None:
            with open(output, 'w') as f:
                js = json.dumps(self.accounts,indent=4)
                f.write(js)
    ```
=== "Example"

    ```python title="example.py" linenums="1"
    from medusa import Medusa
    med = Medusa("users.txt", "wordlist.txt", "ssh", 22)
    med.Run("10.10.0.2")
    med.As_JSON("accounts.json")
    ```

=== "Download"
    [![GitHub](https://img.shields.io/badge/Source-GitHub-blue)](https://github.com/MustafaAbdulazizHamza/packet-lab/blob/master/Python/SubProcess/medusa.py)
