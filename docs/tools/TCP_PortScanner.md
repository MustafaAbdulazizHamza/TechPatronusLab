# TCP_PortScanner

=== "Description"
    A simple TCP SYN port scanner that uses threading to speed up the scanning process.  
=== "Code"
    ```python title="TCP_PortScanner.py" linenums="1"
    from scapy.all import IP, TCP, RandShort, sr1, send
    from concurrent.futures import ThreadPoolExecutor
    from rich.table import Table
    from rich.console import Console
    
    class Scanner:
        def __init__(self, target: str, ports: str) -> None:
            self._target = target
            if '-' in ports:
                self._ports = self._port_range(ports)
            elif ',' in ports:
                self._ports = [int(port.strip()) for port in ports.split(",")]
            else:
                self._ports = [int(ports)]
    
        def _port_range(self, ports: str):
            start, end = map(int, ports.split('-'))
            return list(range(start, end + 1))
    
        def _scan(self, port: int):
            pkt = IP(dst=self._target)/TCP(sport=RandShort(), dport=port, flags="S")
            resp = sr1(pkt, timeout=2, verbose=False)
    
            if resp is None:
                return [port, "Filtered or No Response"]
            elif resp.haslayer(TCP):
                flag = resp[TCP].flags
                if flag == 0x12:
                    send(IP(dst=self._target)/TCP(dport=port, sport=RandShort(), flags="R"), verbose=False)
                    return [port, "Open"]
                elif flag == 0x14:
                    return [port, "Closed"]
            return [port, "Unknown"]
    
        def scan(self):
            console = Console()
            table = Table(show_header=True, header_style="none", show_lines=False)
            table.add_column("Port", justify="right", style="none")
            table.add_column("Status", justify="left", style="none")
    
            with ThreadPoolExecutor() as executor:
                results = executor.map(self._scan, self._ports)
                for port, status in results:
                    table.add_row(str(port), status)
            console.print(table)
    ```
=== "Example"
    ```title="scanner.py"
    from TCP_PortScanner import Scanner
    scanner = Scanner("192.168.1.1", "20-25")
    scanner.scan()
    ```
=== "Download"
    [![GitHub](https://img.shields.io/badge/Source-GitHub-blue)](https://github.com/MustafaAbdulazizHamza/packet-lab/blob/master/Python/Scapy/TCP_PortScanner.py)